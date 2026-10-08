# tests

Every acceptance criterion in [PRD §17](../docs/PRD_v1.7.md) gets at least one automated test. This map says which module owns it.

| AC | What it proves | Owning module | Test type |
|---|---|---|---|
| AC-1 | Capture-ready in ≤60 min, push + MFA set up | `auth`, `app` | end-to-end |
| AC-2 | Fair-housing phrase sets: 100% blocked, ≤10% false positives | `compliance` | unit (phrase suites) |
| AC-3 | Meta lead form → lead + draft in 5 min | `ingest`, `jobs` | integration |
| AC-4 | "Do you buy houses" comment → direct-offer lead with disclosure | `ingest`, `classify`, `drafting` | integration |
| AC-5 | No send without valid hash approval (API, worker, agent paths) | `approvals`, `sender` | unit + integration |
| AC-6 | Duplicate webhook → one lead, one timeline entry | `ingest` | integration |
| AC-7 | Accommodation request → escalation, no refusal | `classify`, `drafting` | integration |
| AC-8 | Investor message → hold list, no score or sequence | `classify`, `scoring` | integration |
| AC-9 | Buyer/tenant stage uses the standard template | `sequences`, `drafting` | integration |
| AC-10 | Morning report at 07:30 Chicago, across DST and Asia/Kolkata | `reports`, `jobs` | unit (time) |
| AC-11 | 2-min video searchable in 5 min; kit with keyword + router | `media`, `content` | integration |
| AC-12 | Footer + unsubscribe; unsubscribe blocks all future sends | `compliance`, `sender` | integration |
| AC-13 | Unreviewed rule pack blocks seller-offer approval | `compliance`, `approvals` | unit |
| AC-14 | LLM down → rules still create the lead, Agent alerted | `classify`, `jobs` | integration |
| AC-15 | Kill switch halts all sends ≤60 s | `sender` | integration |
| AC-16 | Lead magnet sends only after double opt-in | `sequences`, `sender` | integration |
| AC-17 | Reply stops all sequences ≤5 min | `sequences` | integration |
| AC-18 | Bulk to `consent_unknown` refused + flagged | `sender` | unit |
| AC-19a | Failed brief can't move to Film | `content`, `compliance` | unit |
| AC-19b | Unposted approved kit → stuck content after 7 days | `content`, `reports` | unit |
| AC-20 | Helper can't approve or send by any path | `auth`, `approvals`, `sender` | integration |
| AC-21 | Reply-window alert at T-2 h; no send after expiry | `jobs`, `sender` | integration |
| AC-22 | Keyword / DM resolves to the right source piece | `attribution` | unit |
| AC-23 | Re-permission "yes" writes opt-in; silence stays 1:1-only | `sequences` | integration |

*Placeholder: no tests yet.*
