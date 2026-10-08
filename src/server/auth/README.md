# `auth`

Logins for the two V1 roles (Agent, Helper), MFA, sessions, and the workspace/role context used by row-level security.

**PRD:** §02, §12 Security, §13  
**Owns tables:** `user`, `workspace`  
**Acceptance criteria:** AC-1, AC-20

**Must never**
- Grant approval rights before MFA is enabled
- Grant the Helper approve or send rights

*Placeholder: no code yet.*
