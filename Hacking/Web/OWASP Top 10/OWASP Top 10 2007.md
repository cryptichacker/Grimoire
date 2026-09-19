---
tags: [hacking, web, owasp, owasp-2007]
type: owasp-edition
source: https://owasp.org/www-project-top-ten/
last-verified: 2026-08-25
---

# OWASP Top 10 2007

## Up
- [[OWASP Top 10]]

Reoriented around specific attack classes; notably promoted XSS to #1 and introduced CSRF to the list.

| # | Category | Summary |
|---|---|---|
| A1 | Cross-Site Scripting (XSS) | Script injection executed in other users' browsers — the most prevalent flaw of the era. |
| A2 | Injection Flaws | Untrusted data interpreted as a command (SQL, OS, LDAP). |
| A3 | Malicious File Execution | Remote file inclusion / unsafe handling of file references and uploads leading to code execution. |
| A4 | Insecure Direct Object Reference | Exposed internal object identifiers (files, keys, records) that can be tampered to access others' data. |
| A5 | Cross-Site Request Forgery (CSRF) | Forcing a logged-in victim's browser to send unintended state-changing requests. |
| A6 | Information Leakage and Improper Error Handling | Errors and responses that disclose sensitive internal details. |
| A7 | Broken Authentication and Session Management | Weak credential/session handling enabling impersonation. |
| A8 | Insecure Cryptographic Storage | Sensitive data stored with weak or missing encryption. |
| A9 | Insecure Communications | Sensitive traffic sent without (or with weak) transport encryption. |
| A10 | Failure to Restrict URL Access | Sensitive URLs/functions protected only by obscurity, reachable by forced browsing. |
