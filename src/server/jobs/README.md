# `jobs`

Scheduled and retryable work, run by Supabase Cron (pg_cron). Failed work (e.g., a Meta lead fetch) is written to a `jobs` table and retried with backoff.

**PRD:** §05 T3, §12 Scheduled jobs  
**Owns tables:** `jobs`  
**Acceptance criteria:** AC-10, AC-14

Planned schedules: YouTube poll and reply-window watch (15 min); jobs-table retries (5 min); nightly sequences; 06:30 attribution + spend; 07:30 morning report; Monday 06:30 weekly briefs. Each cron entry calls a protected API route or database function that runs the domain module.

**Must never**
- Hold business logic; jobs call domain modules

*Placeholder: no code yet.*
