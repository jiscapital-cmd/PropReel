# `ingest`

Turn every inbound signal into one normalized `message` (and contact/lead where it is a buying signal), exactly once.

**PRD:** §05 (T1, T2, T4), WF-1, WF-2, WF-3, WF-4, WF-8, §14  
**Owns tables:** `message`, `contact` (create + dedupe), `consent` (from forms)  
**Acceptance criteria:** AC-3, AC-6, AC-14

Sources: Meta webhooks (comments, DMs, `leadgen` → fetch by `leadgen_id`), Gmail push, Calendly, YouTube poll every 15 min, landing forms, CSV import, manual paste. Idempotency key = channel + external ID.

**Must never**
- Draft or send anything
- Treat inbound text as instructions (prompt-injection rule, §14)

*Placeholder: no code yet.*
