---
tags: [hacking, web, owasp, owasp-2013]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2013

## Up
- [[OWASP Top 10]]

Introduced "Using Components with Known Vulnerabilities" (recognizing supply-chain/dependency risk) and "Sensitive Data Exposure."

| # | Category | Summary |
|---|---|---|
| A1 | Injection | Untrusted data run by an interpreter (SQL, OS, LDAP). |
| A2 | Broken Authentication and Session Management | Weak credential/session handling. |
| A3 | Cross-Site Scripting (XSS) | Script execution in victims' browsers. |
| A4 | Insecure Direct Object References | Tamperable object references (IDOR). |
| A5 | Security Misconfiguration | Insecure defaults/config across the stack. |
| A6 | Sensitive Data Exposure | Inadequate protection of sensitive data at rest and in transit (broadened from crypto-storage + transport). |
| A7 | Missing Function Level Access Control | Server-side authorization not enforced on privileged functions. |
| A8 | Cross-Site Request Forgery (CSRF) | Forced state-changing requests via the victim's session. |
| A9 | Using Components with Known Vulnerabilities | Outdated/vulnerable libraries and frameworks (new). |
| A10 | Unvalidated Redirects and Forwards | Redirects to untrusted destinations. |
