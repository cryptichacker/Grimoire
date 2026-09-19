---
tags: [hacking, web, owasp, owasp-2021]
type: owasp-edition
source: https://owasp.org/Top10/
last-verified: 2026-08-25
---

# OWASP Top 10 2021

## Up
- [[OWASP Top 10]]

Restructured around **root causes / CWE groups** (numbered `A01…A10`). Broken Access Control rose to #1; new categories: Insecure Design, Software and Data Integrity Failures, and SSRF (added by industry survey).

| # | Category | Summary | PortSwigger |
|---|---|---|---|
| A01 | Broken Access Control | Users act outside their permissions (IDOR, privilege escalation, missing checks). Moved to #1. | [[Access control]] |
| A02 | Cryptographic Failures | Weak/missing cryptography exposing sensitive data (renamed from "Sensitive Data Exposure" to name the root cause). | [[Information disclosure]] |
| A03 | Injection | Untrusted input interpreted as a command/query; XSS now folded in here. | [[SQL injection]], [[Cross-site scripting (XSS)]] |
| A04 | Insecure Design | Flaws in design/architecture that no implementation fix can cover — missing threat modeling and secure-by-design controls (new). | [[Business logic vulnerabilities]] |
| A05 | Security Misconfiguration | Insecure config/defaults; XXE folded in here. | [[XXE injection]] |
| A06 | Vulnerable and Outdated Components | Using components with known vulnerabilities / poor patch management. | |
| A07 | Identification and Authentication Failures | Weak auth, session, and credential handling (renamed from "Broken Authentication"). | [[Authentication]], [[JWT attacks]] |
| A08 | Software and Data Integrity Failures | Trusting code/data without integrity verification (insecure deserialization, unsigned updates, CI/CD compromise) (new). | [[Insecure deserialization]] |
| A09 | Security Logging and Monitoring Failures | Insufficient detection/response to attacks. | |
| A10 | Server-Side Request Forgery (SSRF) | Server coerced into making requests to unintended locations (new, added via community survey). | [[Server-side request forgery (SSRF)]] |
