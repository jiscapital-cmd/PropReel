# `classify`

Decide `lead_type` and `intent` for each message: keyword rules first, then Claude Haiku 5.5 (`claude-haiku-5-5`).

**PRD:** §09 Classification  
**Owns tables:** `lead` (lead_type, intent)  
**Acceptance criteria:** AC-4, AC-7, AC-8, AC-14

Includes the labelled eval set (≥200 messages) used to calibrate the review threshold. Rules keep working when the LLM is down.

**Must never**
- Use protected characteristics or proxies
- Auto-draft anything below the calibrated confidence threshold

*Placeholder: no code yet.*
