# web

Next.js PWA, mobile first. UI only: every read and write goes through the FastAPI backend.

**PRD:** §11

| Screen | Purpose |
|---|---|
| Today | Morning report, hot leads, approvals waiting, one recommended action (default screen) |
| UGC Studio | Reel kanban (Prep → Film/Edit → Kit → Post → Measure), brief editor, prompt library, teleprompter |
| Post Kits | Per-platform captions, SRT, keyword CTA, collaborators, compliance panel, mark as posted |
| Inbox + Approval Queue | Comments, DMs, emails, forms; reply-window countdowns; 1:1 and template approvals; escalations; kill switch |
| Leads + Pipelines | Sellers (listing / direct offer), buyers/tenants, investor hold; timelines and grades |
| Settings | Connections, users and roles, push setup, brand voice, properties, compliance center, budgets |

Web push for hot-lead alerts. On iOS, push works only after the app is added to the Home Screen; onboarding covers this.

**Must never**
- Hold API keys or send logic
- Show approve or send controls to the Helper role

*Placeholder: no code yet.*
