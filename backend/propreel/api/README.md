# `api`

HTTP surface for the PWA and for inbound webhooks. Thin routes that validate input and hand off to domain modules.

**PRD:** §11, §12  
**Owns tables:** none (delegates to domain modules)  
**Acceptance criteria:** entry point for AC-3, AC-4, AC-16

Routes (planned): `/webhooks/meta`, `/webhooks/gmail`, `/webhooks/calendly`, `/webhooks/email-events`, `/leads`, `/approvals`, `/content`, `/kits`, `/r/{slug}` (link-in-bio router), `/reports/today`, `/settings`, `/inngest`.

**Must never**
- Contain business logic
- Call `sender` adapters directly; sends go through `sender.send()` only

*Placeholder: no code yet.*
