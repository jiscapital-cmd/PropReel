# `jobs`

Inngest functions: the durable, retryable work behind every workflow.

**PRD:** §05 T3, §12 Backend  
**Owns tables:** none  
**Acceptance criteria:** AC-10, AC-14

Planned functions: `message.received` → classify → score → draft; `leadgen.fetch` with 24 h retry; YouTube poll (15 min); reply-window watch; nightly sequences; 06:30 attribution + spend; 07:30 report; Monday 06:30 weekly briefs.

**Must never**
- Hold business logic; jobs call domain modules

*Placeholder: no code yet.*
