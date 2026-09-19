---
tags: [hacking, web, portswigger, host-header]
type: vulnerability-category
source: https://portswigger.net/web-security/host-header
last-verified: 2026-08-25
---

# HTTP Host header attacks

## Up
- [[PortSwigger]]

## What it is
The `Host` header (mandatory in HTTP/1.1) tells the server which virtual host the request is for. Because it's fully user-controllable but often *assumed immutable*, apps that use it unsafely — to build absolute URLs, make routing/security decisions, or trust it in back-end logic — become exploitable.

## Testing
- Supply an **arbitrary Host** and see if the request still works (weak validation).
- Inject a **duplicate Host** header, or an **absolute URL** in the request line (`GET https://real/... HTTP/1.1` + a different Host).
- Try `X-Forwarded-Host` (and `X-Host`, `X-Forwarded-Server`) — many apps trust it over `Host`.
- Add indentation/line-wrapping or a port to slip past validation.

## Exploits
- **Password reset poisoning** — if the reset link's domain is built from `Host`/`X-Forwarded-Host`, set it to your server so the victim's token is sent to you.
- **Web cache poisoning** — a reflected/absolute-URL response keyed under a normal cache key but pointing at your Host.
- **Routing-based SSRF** — the front-end routes based on `Host`; set it to an internal hostname/IP to reach internal services (cloud metadata, admin panels).
- **Authentication bypass** — internal-only functionality gated by `Host` (e.g. `Host: localhost` / `intranet`) unlocked by spoofing it.
- **Virtual-host brute-forcing** — enumerate internal vhosts by cycling the `Host` value.

## Prevention
- Prefer **relative URLs**; build absolute URLs from a **configured** trusted domain, not the request `Host`.
- **Validate** `Host` against an allow-list and reject mismatches.
- Don't support `X-Forwarded-Host` etc. unless needed; strip them at the edge.
- Keep internal vhosts off public-facing infrastructure.

## Labs
7 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Basic password reset poisoning | Apprentice | Set `Host` to your exploit server; capture the victim's reset token. | |
| Host header authentication bypass | Practitioner | Spoof `Host` to reach the internal-only admin panel. | |
| Web cache poisoning via ambiguous requests | Practitioner | Duplicate Host / absolute URL to poison the cache. | |
| Routing-based SSRF | Practitioner | Point `Host` at an internal IP the front-end routes to. | |
| SSRF via flawed request parsing | Practitioner | Absolute request-line URL + arbitrary Host to reach internal service. | |
| Host validation bypass via connection state attack | Expert | Reuse a connection whose first request validated, then smuggle a bad Host. | |
| Password reset poisoning via dangling markup | Practitioner | Inject dangling markup into the reset email to exfil the token. | |
