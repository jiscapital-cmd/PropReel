# PropReel

A UGC-to-lead engine for one licensed real estate agent. It turns phone-shot reels, Meta lead ads and email into qualified seller and buyer/tenant conversations, shows which piece of content produced each lead, drafts every reply in the Agent's voice, and sends nothing she hasn't approved.

**Status:** pre-build. This repo holds the product docs and the planned module layout. There is no application code yet.

**Pilot:** one agent, Houston, TX (Texas rule pack), 90 days from Day 0 (the first day lead capture is live).

## Docs
| File | What it is |
|---|---|
| [docs/PRD_v1.6.md](docs/PRD_v1.6.md) | Product requirements, V1 scope, acceptance criteria AC-1 to AC-23 |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Module map, dependency rules, request flows |
| [docs/Reel_Playbook.md](docs/Reel_Playbook.md) | The seven weekly reel formats, hooks, CTAs, compliance notes |
| [docs/benchmarks/producer_benchmark.csv](docs/benchmarks/producer_benchmark.csv) | Reel and YouTube data for five Houston producers (7 Oct 2026) |

## Stack
- **Backend:** Python 3.12, FastAPI, SQLAlchemy 2 + Alembic, Postgres 16 + pgvector
- **Jobs:** Inngest (Python SDK)
- **LLM:** Anthropic Python SDK. Claude Haiku 4.5 classifies; Claude Sonnet 5.5 drafts.
- **Frontend:** Next.js PWA, UI only, calling the API
- **Email:** Gmail API for 1:1, Postmark/Resend for bulk from an authenticated subdomain

## Layout
```
backend/propreel/   FastAPI service, one folder per domain module
web/                Next.js PWA (six screens)
tests/              acceptance-criteria test map
docs/               PRD, architecture, playbook, benchmark data
```

## Rules that shape the code
- Only `backend/propreel/sender` can send anything, and only with a valid approval.
- LLM-facing modules (`classify`, `drafting`) cannot import `sender`.
- Fair-housing and claims checks block approval; the only fix is editing the text.
- Nothing publishes to social platforms. The Agent posts by hand.

*Compliance features flag risk; nothing here is legal advice. The Texas rule pack is a placeholder until reviewed by the Agent's lawyer.*
