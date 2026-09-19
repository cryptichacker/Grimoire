---
tags: [hacking, web, owasp, owasp-2004]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2004

## Up
- [[OWASP Top 10]]

Second edition — a refinement of 2003, and the one many treat as the first widely-adopted release.

| # | Category | Summary |
|---|---|---|
| A1 | Unvalidated Input | Request data not validated before use, enabling injection and logic abuse (renamed from 2003's "Unvalidated Parameters"). |
| A2 | Broken Access Control | Authorization checks not properly enforced across functions and data. |
| A3 | Broken Authentication and Session Management | Flaws in credential/session handling allowing account takeover. |
| A4 | Cross-Site Scripting (XSS) Flaws | Injection of script executed in victims' browsers. |
| A5 | Buffer Overflows | Memory-safety flaws in components triggered by malformed input. |
| A6 | Injection Flaws | User data passed into an interpreter (SQL, OS, LDAP) as part of a command (generalized from "Command Injection"). |
| A7 | Improper Error Handling | Errors that leak information or fail insecurely. |
| A8 | Insecure Storage | Sensitive data stored without adequate encryption/protection. |
| A9 | Application Denial of Service | Application-level resource exhaustion making the service unavailable. |
| A10 | Insecure Configuration Management | Insecure server/framework configuration and defaults. |
