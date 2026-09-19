---
tags: [hacking, web, owasp, owasp-2010]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2010

## Up
- [[OWASP Top 10]]

First edition to rank strictly by **risk** (likelihood × impact) rather than prevalence alone. Injection takes #1; adds "Unvalidated Redirects and Forwards."

| # | Category | Summary |
|---|---|---|
| A1 | Injection | Untrusted data interpreted as part of a command/query (SQL, OS, LDAP). |
| A2 | Cross-Site Scripting (XSS) | Script executed in victims' browsers. |
| A3 | Broken Authentication and Session Management | Flawed credential/session handling. |
| A4 | Insecure Direct Object References | Tamperable references to internal objects (IDOR). |
| A5 | Cross-Site Request Forgery (CSRF) | Unintended state-changing requests via a victim's session. |
| A6 | Security Misconfiguration | Insecure configuration across the app stack (new consolidated category). |
| A7 | Insecure Cryptographic Storage | Sensitive data at rest inadequately protected. |
| A8 | Failure to Restrict URL Access | Missing function-level authorization on sensitive URLs. |
| A9 | Insufficient Transport Layer Protection | Weak/absent TLS for sensitive traffic. |
| A10 | Unvalidated Redirects and Forwards | Redirects/forwards to attacker-controlled destinations (phishing, control bypass). |
