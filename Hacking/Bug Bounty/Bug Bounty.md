---
tags: [hacking, bug-bounty]
type: moc
---

# Bug Bounty

## Up
- [[Hacking]]

Bug bounty study hub — a growing library of **real, publicly disclosed reports** organized by vulnerability class, plus the **exploitation patterns** distilled from them. A daily task appends ~10 fresh disclosed reports (≈2 per category) to the running logs under Reports.

> All reports here are **publicly disclosed** (HackerOne, Bugcrowd, Intigriti, YesWeHack, GitHub, blogs). Study them for patterns; apply only to authorized targets / in-scope programs.

## Subtopics
- [[Reports]] — disclosed-report logs by vulnerability + the [[Pattern]] page

## Vulnerability focus
1. [[IDOR]] — Insecure Direct Object References
2. [[Injection]] — split: [[SQLi]] · [[XSS]] · [[SSTI]] · [[RCE]]
3. [[Security Misconfigurations]] — CORS, defaults, headers, exposed config
4. [[Broken Authentication]] — auth & session management flaws
5. [[Exposed APIs & Info Disclosure]] — leaked keys, PII, secrets

## Related
- [[Web]] (PortSwigger/OWASP) — deep technique theory behind these bugs
- [[CTF]] · [[HTTP (80,443)]] — hands-on web exploitation
- [[Methodology]] — PTES/ATT&CK framing · [[CVSS]] for severity

---

## How this library is maintained
- **Daily cron (09:00 UTC):** researches ~10 newly/notably disclosed reports across the 5 categories, de-duplicates against existing entries, and appends dated items to each [[Reports]] log; enriches [[Pattern]] when a new technique shows up.
- Each entry records: **title · source link · program · bounty · summary · technique/pattern · takeaway.**
