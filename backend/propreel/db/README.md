# `db`

SQLAlchemy 2 models and Alembic migrations for the 16 core tables.

**PRD:** §12a  
**Owns tables:** all 16 tables  
**Acceptance criteria:** supports all

Tables: workspace, user, contact, consent, suppression, lead, message, draft, approval, send, property, content_piece, post_kit, sequence_enrollment, prompt_template, audit_event. Append-only rules enforced in the database, not just by convention.

**Must never**
- Allow UPDATE or DELETE on append-only tables (`consent`, `audit_event`, approval history)

*Placeholder: no code yet.*
