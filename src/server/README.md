# src/server

Server-only domain modules for the Next.js app. Nothing here is ever bundled into the browser, and all API keys live here. Each module folder has a README with its purpose, PRD sections, tables, acceptance criteria and what it must never do.

| Module | Job |
|---|---|
| `ingest` | Normalize inbound signals, dedupe, idempotency |
| `classify` | Lead type + intent (rules, then Claude Haiku 5.5) |
| `scoring` | A–D grades from allowed inputs |
| `drafting` | Replies, briefs, captions (Claude Sonnet 5.5) |
| `compliance` | Fair housing, claims, disclosures, rule packs |
| `approvals` | 1:1 and template-segment approval hashes |
| `sender` | The only egress; ten checks, channel adapters (Gmail, Resend, Meta, YouTube) |
| `sequences` | Nurture, re-engagement, re-permission |
| `content` | Reel pipeline, post kits, keywords, link-in-bio |
| `media` | Upload, Groq Whisper transcription, tags, search |
| `attribution` | First/last touch, spend |
| `reports` | Morning report, alerts, diagnostics |
| `jobs` | Supabase Cron (pg_cron) schedules and the retry jobs table |
| `auth` | Agent/Helper roles, MFA |
| `db` | Supabase schema, migrations, row-level security |

HTTP entry points live in [../app/api](../app/api). Dependency rules are in [../../docs/ARCHITECTURE.md](../../docs/ARCHITECTURE.md).

*Placeholder: no code yet.*
