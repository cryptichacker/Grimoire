---
tags: [hacking, web, owasp, owasp-2017]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2017

## Up
- [[OWASP Top 10]]

Added XXE, Insecure Deserialization, and Insufficient Logging & Monitoring; merged access-control items; dropped CSRF and unvalidated redirects (largely mitigated by frameworks).

| # | Category | Summary | PortSwigger |
|---|---|---|---|
| A1 | Injection | Untrusted data interpreted as a command/query. | [[SQL injection]], [[NoSQL injection]], [[Command injection]] |
| A2 | Broken Authentication | Weak credential/session management enabling takeover. | [[Authentication]] |
| A3 | Sensitive Data Exposure | Sensitive data insufficiently protected at rest/in transit. | [[Information disclosure]] |
| A4 | XML External Entities (XXE) | XML parsers resolving attacker-defined external entities. | [[XXE injection]] |
| A5 | Broken Access Control | Authorization not enforced (merged IDOR + function-level access). | [[Access control]] |
| A6 | Security Misconfiguration | Insecure defaults/config across the stack. | |
| A7 | Cross-Site Scripting (XSS) | Script executed in victims' browsers. | [[Cross-site scripting (XSS)]] |
| A8 | Insecure Deserialization | Untrusted serialized data deserialized unsafely (→ RCE). | [[Insecure deserialization]] |
| A9 | Using Components with Known Vulnerabilities | Outdated/vulnerable dependencies. | |
| A10 | Insufficient Logging & Monitoring | Attacks undetected due to poor logging/alerting. | |
