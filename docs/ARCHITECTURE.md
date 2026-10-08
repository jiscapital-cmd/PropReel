# PropReel Architecture

Companion to [PRD v1.8](PRD_v1.8.md) §12. Describes the planned module layout; no code exists yet.

## Stack
| Layer | Choice |
|---|---|
| App | One full-stack Next.js app (TypeScript): PWA screens in `src/app`, server modules in `src/server` |
| API | Next.js API routes in `src/app/api`: app API for the PWA and webhook receivers |
| Scheduled jobs | Supabase Cron (pg_cron), plus a jobs table for retries (`src/server/jobs`) |
| LLM | Anthropic TypeScript SDK: Claude Haiku 5.5 `claude-haiku-5-5` (classify), Claude Sonnet 5.5 `claude-sonnet-5-5` (drafts, briefs) |
| Data | Supabase (Postgres), SQL migrations, pgvector, row-level security (`src/server/db`) |
| Media | S3/R2 signed uploads, FFmpeg preview, Groq Whisper API transcription |
| Email | Gmail API (1:1), Resend (bulk + transactional, authenticated subdomain) |

## Layers
```
src/app (PWA screens)
   │  same app, session cookie, MFA
   ▼
src/app/api (routes) ─────────► src/server/jobs (Supabase Cron (pg_cron) + jobs table)
   │                                  │
   ▼                                  ▼
ingest → classify → scoring → drafting → compliance → [Agent approves] → approvals
                                                                            │
content · media · sequences · attribution · reports                         ▼
                                                                         sender ──► Gmail / Resend / Meta / YouTube
   all modules read/write through db (Supabase)     audit_event written by every decision
```

## Dependency rules
1. **`sender` is the only egress.** No other module imports a channel adapter.
2. **LLM-facing modules cannot reach `sender`.** `classify` and `drafting` must not import `sender` or `approvals`. Enforce with an ESLint import-restriction rule in CI.
3. **`src/server` is server-only.** Every module marks itself server-only so it can never be bundled into the browser; API keys live only there.
4. **API routes and jobs hold no business logic.** They call domain modules.
5. **Domain modules don't touch each other's tables directly.** Each module owns its tables (see each README) and exposes functions.
6. **Append-only tables** (`consent`, `audit_event`, approval history) are protected in the database (no UPDATE/DELETE grants, or triggers), not only in code.
7. **Inbound text is data.** Nothing from a comment, DM or email is executed as an instruction.

## Main flows

**Organic comment → reply (WF-2/3)**
1. The Meta webhook route verifies the signature → `ingest` stores the `message` (idempotent on external ID).
2. `classify` (rules, then Haiku 5.5) → `scoring` → `drafting` (Sonnet 5.5).
3. `compliance` checks the draft; failures stay in the queue for editing.
4. The Agent approves in the PWA → `approvals` stores a 1:1 hash.
5. `sender` runs its ten checks, sends inside the reply window, writes `send` + `audit_event`.
6. `attribution` links the lead to the source piece by keyword or post.

**Meta lead form (WF-1)**
`ingest` receives `leadgen_id` → fetches the full lead; on failure a row goes in the jobs table and cron retries it with backoff for 24 h → same path as above from step 2.

**Nurture (WF-9)**
Nightly cron → `sequences` picks due enrollments → renders the template-segment-approved email → `sender` checks each recipient (consent, suppression, caps, quiet hours) → Resend. Any reply → `sequences` stops all enrollments for that contact within 5 minutes.

**Reel (WF-7)**
Monday 06:30 cron → `drafting` writes 7 briefs → `compliance` pre-check → Agent approves → Agent films and uploads → `media` transcribes with Groq Whisper → `content` builds the kit with a keyword → Agent approves and posts by hand → registers URL → `attribution` and `reports` measure.

## Scheduled jobs (Supabase Cron (pg_cron))
| Schedule | Job |
|---|---|
| Every 15 min | YouTube comment poll; reply-window check (alert at T-2 h) |
| Every 5 min | Retry due rows in the jobs table (e.g., Meta lead fetch) |
| Nightly | Sequence steps (WF-9/10/11) |
| 06:30 market time | Attribution recompute + Meta spend pull |
| 07:30 market time | Morning report |
| Monday 06:30 | Weekly reel briefs |

## Open design questions
- **Webhook response time.** PRD §12 runs classify → score → draft from the webhook's API route. Meta expects a fast reply to webhooks, and the LLM steps may exceed that on serverless hosting. Recommended: the route stores the message and returns at once; processing runs from the jobs table. Decide before building `ingest`.
- **FFmpeg on serverless hosting.** The preview step may need to run on the phone at upload or in a small separate worker.

## Module map
| Module | Location | PRD | Tables | ACs |
|---|---|---|---|---|
| api | `src/app/api` | §11, §12 | — | entry for AC-3, 4, 16 |
| ingest | `src/server/ingest` | §05, WF-1/2/3/4/8, §14 | message, contact, consent | AC-3, 6, 14 |
| classify | `src/server/classify` | §09 | lead (type, intent) | AC-4, 7, 8, 14 |
| scoring | `src/server/scoring` | §09 | lead (grade) | AC-9 |
| drafting | `src/server/drafting` | §06, §08, §09 | draft, prompt_template | AC-9, 11 |
| compliance | `src/server/compliance` | §10 | — | AC-2, 12, 13, 19a |
| approvals | `src/server/approvals` | §10.10 | approval | AC-5, 20 |
| sender | `src/server/sender` | §10, §13, §14 | send | AC-5, 12, 15, 18, 21 |
| sequences | `src/server/sequences` | WF-9/10/11 | sequence_enrollment | AC-9, 16, 17, 23 |
| content | `src/server/content` | WF-7 | content_piece, post_kit | AC-11, 19a, 19b, 22 |
| media | `src/server/media` | §12 | content_piece (media) | AC-11 |
| attribution | `src/server/attribution` | §09, §16 | lead (touches) | AC-22 |
| reports | `src/server/reports` | §11, §16 | — | AC-10, 19b |
| jobs | `src/server/jobs` | §05 T3, §12 | jobs table | AC-10, 14 |
| auth | `src/server/auth` | §02, §13 | user, workspace | AC-1, 20 |
| db | `src/server/db` | §12a | all core tables | — |

## Build order
Follows PRD §18: foundation (db, auth, approvals, sender, manual/CSV capture, reports) → paid leads (Meta ingest, classify, scoring, drafting, compliance) → organic (content, media, attribution, IG/FB) → email program (sequences on a warmed domain).
