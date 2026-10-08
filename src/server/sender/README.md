# `sender`

The only way anything leaves the system. Runs the ten checks in order, then calls a channel adapter.

**PRD:** §10, §12 Sender service, §13, §14  
**Owns tables:** `send`; reads `suppression`, `consent`, `approval`  
**Acceptance criteria:** AC-5, AC-12, AC-15, AC-18, AC-21

Check order: kill switch → role → hash → approval validity → consent → suppression → quiet hours → caps → reply window → send + audit. Adapters: Gmail API (1:1), Resend (bulk + transactional), Meta messaging, YouTube replies. Pauses bulk on bounce >2% or complaints >0.1%.

**Must never**
- Be importable from `classify`, `drafting` or any LLM tool
- Send when any check fails

*Placeholder: no code yet.*
