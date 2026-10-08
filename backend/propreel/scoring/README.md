# `scoring`

Grade seller and buyer/tenant leads A–D from allowed inputs only, with a one-line rationale.

**PRD:** §09 Scoring, Fairness rule  
**Owns tables:** `lead` (grade, grade_rationale)  
**Acceptance criteria:** AC-9 (consistency), hard requirement: same criteria for everyone

Agent overrides are logged with a reason in `audit_event`.

**Must never**
- Read name, photo, profile text, language or person location
- Treat missing data as negative

*Placeholder: no code yet.*
