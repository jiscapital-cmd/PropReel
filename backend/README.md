# backend

FastAPI service (Python 3.12). One package, `propreel`, split into domain modules. Each module folder has a README with its purpose, PRD sections, tables, acceptance criteria and what it must never do.

| Module | Job |
|---|---|
| `api` | HTTP routes for the PWA and webhooks |
| `ingest` | Normalize inbound signals, dedupe, idempotency |
| `classify` | Lead type + intent (rules, then Haiku) |
| `scoring` | A–D grades from allowed inputs |
| `drafting` | Replies, briefs, captions (Sonnet) |
| `compliance` | Fair housing, claims, disclosures, rule packs |
| `approvals` | 1:1 and template-segment approval hashes |
| `sender` | The only egress; ten checks, channel adapters |
| `sequences` | Nurture, re-engagement, re-permission |
| `content` | Reel pipeline, post kits, keywords, link-in-bio |
| `media` | Upload, transcription, tags, search |
| `attribution` | First/last touch, spend |
| `reports` | Morning report, alerts, diagnostics |
| `jobs` | Inngest functions |
| `auth` | Agent/Helper roles, MFA |
| `db` | Models and migrations |

Dependency rules are in [../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md).

*Placeholder: no code yet.*
