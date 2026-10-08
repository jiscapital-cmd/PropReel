# PropReel

A UGC-to-lead engine for one licensed real estate agent. It turns phone-shot reels, Meta lead ads and email into qualified seller and buyer/tenant conversations, shows which piece of content produced each lead, drafts every reply in the Agent's voice, and sends nothing she hasn't approved.

**Status:** pre-build. This repo holds the product docs and the planned module layout. There is no application code yet.

**Pilot:** one agent, Houston, TX (Texas rule pack), 90 days from Day 0 (the first day lead capture is live).

## Docs
| File | What it is |
|---|---|
| [docs/PRD_v1.7.md](docs/PRD_v1.7.md) | Product requirements, V1 scope, acceptance criteria AC-1 to AC-23 |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Module map, dependency rules, request flows |
| [docs/Reel_Playbook.md](docs/Reel_Playbook.md) | The seven weekly reel formats, hooks, CTAs, compliance notes |
| [docs/benchmarks/producer_benchmark.csv](docs/benchmarks/producer_benchmark.csv) | Reel and YouTube data for five Houston producers (7 Oct 2026) |

## Stack
One full-stack Next.js app (TypeScript). No separate backend, no Python.
- **App and API:** Next.js PWA screens and API routes (app API + webhooks)
- **Database and scheduled jobs:** Bolt Database (Postgres, pgvector, row-level security); Bolt Database cron for scheduled work
- **LLM:** Anthropic TypeScript SDK. Claude Haiku 5.5 (`claude-haiku-5-5`) classifies; Claude Sonnet 5.5 (`claude-sonnet-5-5`) drafts.
- **Email:** Gmail API for 1:1 replies; Resend for bulk and transactional email from an authenticated subdomain
- **Transcription:** Groq Whisper API

## Layout
```
src/app/       Next.js screens (six) and API routes
src/server/    server-only domain modules: ingest, classify, scoring, drafting, compliance,
               approvals, sender, sequences, content, media, attribution, reports, jobs, auth, db
tests/         acceptance-criteria test map
docs/          PRD, architecture, playbook, benchmark data
```

## Rules that shape the code
- Only `src/server/sender` can send anything, and only with a valid approval.
- LLM-facing modules (`classify`, `drafting`) cannot import `sender`.
- Code in `src/server` never runs in the browser.
- Fair-housing and claims checks block approval; the only fix is editing the text.
- Nothing publishes to social platforms. The Agent posts by hand.

*Compliance features flag risk; nothing here is legal advice. The Texas rule pack is a placeholder until reviewed by the Agent's lawyer.*
