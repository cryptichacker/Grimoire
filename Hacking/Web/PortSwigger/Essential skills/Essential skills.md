---
tags: [hacking, web, portswigger, essential-skills]
type: vulnerability-category
source: https://portswigger.net/web-security/essential-skills
last-verified: 2026-08-25
---

# Essential skills

## Up
- [[PortSwigger]]

## What it is
Cross-cutting testing skills that apply across every vulnerability class, rather than a single bug type: obfuscating payloads with encodings, using Burp Scanner alongside manual testing, and confirming/exploiting bugs out-of-band.

## Obfuscating attacks using encodings
Filters and WAFs often pattern-match raw payloads; encodings that a downstream system decodes *after* the filter let you slip through. Techniques:
- **URL** and **double URL** encoding (`%2f`, `%252f`).
- **HTML entities** (`&#x53;`, `&apos;`) — decoded by the browser/parser.
- **XML entities** — e.g. to smuggle SQL keywords past a WAF.
- **Unicode / hex / octal escapes** in JS and other contexts.
- **SQL `CHAR()`** to build strings without quotes.
- **Layered (multiple) encodings** where each hop decodes once.
Key idea: exploit **handling discrepancies** between connected systems.

## Using Burp Scanner during manual testing
Augment manual work with targeted automation: scan a **specific request**, define **custom insertion points**, and scan **non-standard data structures** (nested/encoded parameters) so automated checks reach the same inputs you're testing by hand — catching what manual review misses.

## Out-of-band (OAST) techniques
Many bugs are **blind** — no result in the response. Use **Burp Collaborator** to give the target a unique external domain and watch for the **DNS/HTTP interaction** it triggers, confirming (and often exfiltrating through) blind SSRF, blind SQLi, blind OS command injection, blind XXE, etc.

## Mystery labs
The academy's "mystery" challenges present a bug with **no hint** about its class, practising the real-world skill of identifying an unknown vulnerability from behaviour alone.

## Labs
2 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Discovering vulnerabilities quickly with targeted scanning | Practitioner | Scan a single request with a custom insertion point to find the bug fast. | |
| Scanning non-standard data structures | Practitioner | Define insertion points inside an encoded/nested value and scan it. | |
