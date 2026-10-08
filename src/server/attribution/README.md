# `attribution`

Credit leads and appointments to the content piece or campaign that produced them.

**PRD:** §09 Attribution model, §16  
**Owns tables:** `lead` (first_touch_source, last_touch_source)  
**Acceptance criteria:** AC-22, Track A/B metrics

Resolution order: per-piece keyword → comment on a registered post → router/UTM → shared post in DM → Agent tag. Nightly recompute and Meta spend pull (`ads_read`).

**Must never**
- Blend paid and organic results

*Placeholder: no code yet.*
