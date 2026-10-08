# `approvals`

Bind approvals to exact content: 1:1 (content + recipient + channel) and template-segment (template version + segment rules + fields + channel), SHA-256.

**PRD:** §10.10  
**Owns tables:** `approval`  
**Acceptance criteria:** AC-5, AC-20

**Must never**
- Accept an approval from the Helper role
- Keep an approval alive after an edit, TTL expiry (72 h / 30 d) or a linked property change

*Placeholder: no code yet.*
