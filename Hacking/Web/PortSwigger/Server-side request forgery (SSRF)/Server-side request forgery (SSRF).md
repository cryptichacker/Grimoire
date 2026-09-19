---
tags: [hacking, web, portswigger, ssrf]
type: vulnerability-category
source: https://portswigger.net/web-security/ssrf
last-verified: 2026-08-25
---

# Server-side request forgery (SSRF)

## Up
- [[PortSwigger]]

## What it is
SSRF lets an attacker make the server-side application issue requests to an unintended location. It can expose internal services, read cloud metadata, and sometimes lead to remote code execution — with the request appearing to come from the trusted server.

## SSRF against the server itself
Point the request at the loopback interface to reach admin functionality that trusts local connections:
```
POST /product/stock
stockApi=http://localhost/admin
```
Local requests often skip authentication (e.g. an admin panel that trusts `127.0.0.1`).

## SSRF against back-end systems
Reach internal, non-routable hosts the server can see but you can't:
```
stockApi=http://192.168.0.68/admin
```

## Bypassing defenses
**Blacklist filters** (e.g. blocking `127.0.0.1`/`localhost`) — use alternative representations:
- Decimal `2130706433`, octal `017700000001`, shortened `127.1`
- A domain you control that resolves to `127.0.0.1`
- Mixed encodings, redirect via a permitted host

**Whitelist filters** — abuse URL-parsing quirks so the "expected" host is present but not the real target:
- Credentials: `https://expected-host@evil-host`
- Fragment: `https://evil-host#expected-host`
- Subdomain trick: `https://expected-host.evil-host`
- Double URL-encoding of characters the parser and filter treat differently

**Open redirection** — if a whitelisted app has an open redirect, point SSRF at that URL and let it bounce you to the internal target.

## Blind SSRF
No response is returned. Detect via **OAST** — make the server hit a Burp Collaborator domain and watch for the DNS/HTTP interaction. Impact can still be high (e.g. reaching an internal service vulnerable to something like Shellshock via the `User-Agent`).

## Finding hidden attack surface
- Partial URLs (a hostname/path fragment the server completes).
- URLs inside data formats (XML external entities).
- The `Referer` header (analytics software may fetch it).

## Prevention
- Allow-list permitted hosts/URLs (don't blacklist), and re-validate after any redirect.
- Don't send raw responses back to the client.
- Enforce network-layer segmentation so the app server can't reach sensitive internal services or cloud metadata (e.g. block `169.254.169.254`).

## Labs
7 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Basic SSRF against the local server | Apprentice | `stockApi=http://localhost/admin` to reach the admin panel. | |
| Basic SSRF against another back-end system | Apprentice | Scan `192.168.0.x/admin` via the stock API. | |
| Blind SSRF with out-of-band detection | Practitioner | Put a Collaborator URL in the `Referer` and watch for the hit. | |
| SSRF with blacklist-based input filter | Practitioner | Bypass `localhost` block with `127.1`/decimal/encoding + case. | |
| SSRF with filter bypass via open redirection vulnerability | Practitioner | Chain an on-site open redirect to reach `/admin`. | |
| SSRF with whitelist-based input filter | Expert | Use `@`, `#`, and double-encoding to smuggle the real host. | |
| Blind SSRF with Shellshock exploitation | Expert | Blind SSRF to an internal host, Shellshock via `User-Agent`, exfil via Collaborator. | |
