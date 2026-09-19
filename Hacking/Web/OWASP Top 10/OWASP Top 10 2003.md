---
tags: [hacking, web, owasp, owasp-2003]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2003

## Up
- [[OWASP Top 10]]

The inaugural edition (published January 2003) — "The Ten Most Critical Web Application Security Vulnerabilities." A qualitative, expert-curated list rather than a data-driven ranking.

| # | Category | Summary |
|---|---|---|
| A1 | Unvalidated Parameters | Web request data (query, form, cookie, header) used without validation, enabling injection and tampering. |
| A2 | Broken Access Control | Restrictions on what authenticated users may do are not properly enforced. |
| A3 | Broken Account and Session Management | Weak handling of credentials, sessions, and tokens allowing hijacking or impersonation. |
| A4 | Cross-Site Scripting (XSS) Flaws | Attacker-supplied script reflected/stored and executed in other users' browsers. |
| A5 | Buffer Overflows | Unchecked input overruns memory buffers in application components, causing crashes or code execution. |
| A6 | Command Injection Flaws | User input passed into an external command/interpreter (OS, SQL, LDAP), running attacker commands. |
| A7 | Error Handling Problems | Errors not handled cleanly, leaking internal details or leaving the app in an insecure state. |
| A8 | Insecure Use of Cryptography | Weak, homegrown, or misapplied cryptography failing to protect sensitive data. |
| A9 | Remote Administration Flaws | Insecure remote admin interfaces exposing privileged functionality. |
| A10 | Web and Application Server Misconfiguration | Insecure default or ad-hoc server configuration weakening the whole stack. |
