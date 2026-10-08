# PropReel Architecture

Companion to [PRD v1.6](PRD_v1.6.md) §12. Describes the planned module layout; no code exists yet.

## Stack
| Layer | Choice |
|---|---|
| UI | Next.js PWA (`web/`), UI only |
| API | FastAPI, Python 3.12 (`backend/propreel/api`) |
| Jobs | Inngest Python SDK, served from the FastAPI app (`backend/propreel/jobs`) |
| LLM | Anthropic Python SDK: Claude Haiku 4.5 (classify), Claude Sonnet 5.5 (drafts, briefs) |
| Data | Postgres 16 + pgvector; SQLAlchemy 2 + Alembic (`backend/propreel/db`) |
| Media | S3/R2 signed uploads, FFmpeg, Deepgram/Whisper |
| Email | Gmail API (1:1), Postmark/Resend (bulk, authenticated subdomain) |

## Layers
```
web (PWA)
   │  HTTPS (session cookie, MFA)
   ▼
api ───────────────► jobs (Inngest functions)
   │                    │
   ▼                    ▼
ingest → classify → scoring → drafting → compliance → [Agent approves] → approvals
                                                                            │
content · media · sequences · attribution · reports                         ▼
                                                                         sender ──► Gmail / Postmark / Meta / YouTube
   all modules read/write through db (Postgres)          audit_event written by every decision
```

## Dependency rules
1. **`sender` is the only egress.** No other module imports a channel adapter.
2. **LLM-facing modules cannot reach `sender`.** `classify` and `drafting` must not import `sender` or `approvals`. Enforce with an import-linter contract in CI.
3. **`api` and `jobs` hold no business logic.** They call domain modules.
4. **Domain modules don't call each other's tables directly.** Each module owns its tables (see each README) and exposes functions; others call those functions.
5. **Append-only tables** (`consent`, `audit_event`, approval history) are protected in the database (no UPDATE/DELETE grants or triggers), not only in code.
6. **Inbound text is data.** Nothing from a comment, DM or email is executed as an instruction.

## Main flows

**Organic comment → reply (WF-2/3)**
1. `api` receives the Meta webhook → `ingest` stores the `message` (idempotent on external ID).
2. `jobs` fires `message.received` → `classify` (rules, then Haiku) → `scoring` → `drafting`.
3. `compliance` checks the draft; failures stay in the queue for editing.
4. The Agent approves in the PWA → `approvals` stores a 1:1 hash.
5. `sender` runs its ten checks, sends inside the reply window, writes `send` + `audit_event`.
6. `attribution` links the lead to the source piece by keyword or post.

**Meta lead form (WF-1)**
`ingest` receives `leadgen_id` → `jobs` fetches the full lead (retry 24 h) → same path as above from step 2.

**Nurture (WF-9)**
Nightly `jobs` → `sequences` picks due enrollments → renders the template-segment-approved email → `sender` checks each recipient (consent, suppression, caps, quiet hours) → send. Any reply → `sequences` stops all enrollments for that contact within 5 minutes.

**Reel (WF-7)**
Monday `jobs` → `drafting` writes 7 briefs → `compliance` pre-check → Agent approves → Agent films and uploads → `media` transcribes → `content` builds the kit with a keyword → Agent approves and posts by hand → registers URL → `attribution` and `reports` measure.

## Module map
| Module | PRD | Tables | ACs |
|---|---|---|---|
| api | §11, §12 | — | entry for AC-3, 4, 16 |
| ingest | §05, WF-1/2/3/4/8, §14 | message, contact, consent | AC-3, 6, 14 |
| classify | §09 | lead (type, intent) | AC-4, 7, 8, 14 |
| scoring | §09 | lead (grade) | AC-9 |
| drafting | §06, §08, §09 | draft, prompt_template | AC-9, 11 |
| compliance | §10 | — | AC-2, 12, 13, 19a |
| approvals | §10.10 | approval | AC-5, 20 |
| sender | §10, §13, §14 | send | AC-5, 12, 15, 18, 21 |
| sequences | WF-9/10/11 | sequence_enrollment | AC-9, 16, 17, 23 |
| content | WF-7 | content_piece, post_kit | AC-11, 19a, 19b, 22 |
| media | §12 | content_piece (media) | AC-11 |
| attribution | §09, §16 | lead (touches) | AC-22 |
| reports | §11, §16 | — | AC-10, 19b |
| jobs | §05 T3 | — | AC-10, 14 |
| auth | §02, §13 | user, workspace | AC-1, 20 |
| db | §12a | all 16 | — |

## Build order
Follows PRD §18: foundation (db, auth, approvals, sender, ingest manual/CSV, reports) → paid leads (Meta ingest, classify, scoring, drafting, compliance) → organic (content, media, attribution, IG/FB) → email program (sequences on a warmed domain).
