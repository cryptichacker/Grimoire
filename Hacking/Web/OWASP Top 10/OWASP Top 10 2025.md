---
tags: [hacking, web, owasp, owasp-2025]
type: owasp-edition
source: https://owasp.org/Top10/2025/
last-verified: 2026-08-25
---

# OWASP Top 10 2025

## Up
- [[OWASP Top 10]]

Current edition. Broken Access Control stays #1 (SSRF folded into it); Security Misconfiguration jumps to #2. Two new categories: **Software Supply Chain Failures** and **Mishandling of Exceptional Conditions**.

| # | Category | Summary | PortSwigger |
|---|---|---|---|
| A01 | Broken Access Control | Still #1. Now absorbs **SSRF** as a manifestation of improper access control. | [[Access control]], [[Server-side request forgery (SSRF)]] |
| A02 | Security Misconfiguration | Up from #5 — reflecting cloud/infra complexity and default-config risk. | [[XXE injection]] |
| A03 | Software Supply Chain Failures | **New** — broadens 2021's "Vulnerable and Outdated Components" to the whole ecosystem: dependencies, build/CI-CD, and distribution compromise. | |
| A04 | Cryptographic Failures | Down from #2. Weak/missing cryptography exposing data. | [[Information disclosure]] |
| A05 | Injection | Down from #3. Untrusted input interpreted as a command/query (includes XSS). | [[SQL injection]], [[Cross-site scripting (XSS)]] |
| A06 | Insecure Design | Down from #4. Design/architecture flaws needing secure-by-design controls. | [[Business logic vulnerabilities]] |
| A07 | Authentication Failures | Renamed from "Identification and Authentication Failures." Weak auth/session/credential handling. | [[Authentication]], [[JWT attacks]] |
| A08 | Software or Data Integrity Failures | Trusting code/data without integrity checks (deserialization, unsigned updates, pipeline tampering). | [[Insecure deserialization]] |
| A09 | Security Logging and Alerting Failures | Renamed to stress **alerting** — not just logging, but acting on it. | |
| A10 | Mishandling of Exceptional Conditions | **New** — improper error handling, logic errors, and "failing open" (insecure default behaviour on error). | |
