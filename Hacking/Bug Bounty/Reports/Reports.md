---
tags: [hacking, bug-bounty, reports]
type: moc
---

# Reports

## Up
- [[Bug Bounty]]

Running logs of publicly disclosed bug-bounty reports, one per vulnerability class. New entries are **appended** daily (newest at the top of each log's Reports section).

## Logs
- [[IDOR]]
- [[Injection]] — split by type: [[SQLi]] · [[XSS]] · [[SSTI]] · [[RCE]]
- [[Security Misconfigurations]]
- [[Broken Authentication]]
- [[Exposed APIs & Info Disclosure]]

## Distilled knowledge
- [[Pattern]] — the exploitation patterns/methodology used across these reports (enriched as new techniques appear)

---

### Entry format
```
### YYYY-MM-DD — <Title> (<Program>) — <bounty>
- Source: [platform #id](url)
- Type: <subtype>
- Summary: what the bug was
- Technique / pattern: how it was found & exploited
- Takeaway: the reusable lesson
```
