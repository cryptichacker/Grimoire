---
tags: [hacking, web, portswigger, web-cache-deception]
type: vulnerability-category
source: https://portswigger.net/web-security/web-cache-deception
last-verified: 2026-08-25
---

# Web cache deception

## Up
- [[PortSwigger]]

## What it is
Web cache deception tricks a web cache into storing sensitive, dynamic content (belonging to a victim) by exploiting discrepancies between how the **cache** and the **origin server** interpret a URL. The attacker lures the victim to a crafted URL; the cache stores the victim's private response; the attacker then requests the same URL and reads it.

Distinct from **web cache poisoning**: poisoning manipulates the cache *key* to serve malicious content to others; deception abuses *cache rules* to store sensitive content the attacker retrieves.

## How caches decide
- **Cache key** — derived from parts of the request (usually path + query), maybe headers.
- **Cache rules** — what to store, based on: static **file extensions** (`.css`, `.js`), static **directories** (`/static`, `/assets`), or specific **file names** (`robots.txt`, `favicon.ico`).

The attack makes a *dynamic* response look, to the cache, like a *static* one.

## Constructing the attack

**Path mapping discrepancies** — a REST origin may ignore a trailing segment while the cache treats the whole path as a static file:
```
/user/123/profile/wcd.css
```
Origin returns the profile; cache stores it as CSS. Test by appending junk segments (identical response ⇒ origin abstracts the path), then a static extension.

**Delimiter discrepancies** — frameworks use different delimiters (Spring `;`, Rails `.`):
```
/profile;foo.css
```
Spring origin returns `/profile`; a non-Spring cache caches the `.css`. Test candidate delimiters `; : ? # @ & %` by comparing responses.

**Delimiter decoding discrepancies** — one side decodes an encoded delimiter, the other doesn't:
```
/profile%23wcd.css   (%23 = #)
```
Origin decodes and truncates at `#` → returns `/profile`; cache keeps it encoded → caches as `.css`. Also try `%3f` (`?`), `%00`, `%0A`, `%09`.

**Static-directory normalization** — differing dot-segment resolution:
```
/static/..%2fprofile
```
Origin normalizes to `/profile` (data); cache sees the literal path under `/static` and caches it. The reverse (cache normalizes, origin doesn't) needs an added delimiter, e.g. `/profile;%2f%2e%2e%2fstatic`.

**Static file-name rules** — same trick aimed at a cached filename:
```
/profile%2f%2e%2e%2findex.html
```

## Detection
- **Origin normalization**: send `/aaa/..%2fprofile` — if it matches the real profile, the origin decodes+resolves.
- **Cache normalization**: compare caching of `/assets/..%2fjs/file.js` vs `/aaa/..%2fassets/js/file.js`; a `/assets/aaa` still being cached confirms a directory rule.
- Use Param Miner to cache-bust with unique query strings, and Burp Intruder (payload encoding off) to test delimiters at scale.

## Prevention
- Serve dynamic responses with `Cache-Control: no-store, private`.
- Configure the CDN to respect origin `Cache-Control` and enable deception protections (e.g. Cloudflare Cache Deception Armor validating Content-Type vs extension).
- Eliminate path-interpretation discrepancies between cache and origin; normalize consistently.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Exploiting path mapping for web cache deception | Practitioner | Append `/wcd.css` to a REST endpoint so the cache stores the profile/API key. | |
| Exploiting path delimiters for web cache deception | Practitioner | Find the framework delimiter (e.g. `;`) + static extension. | |
| Exploiting origin server normalization for web cache deception | Practitioner | `/static/..%2f<dynamic>` — origin resolves, cache doesn't. | |
| Exploiting cache server normalization for web cache deception | Practitioner | Cache resolves dot-segments; add a delimiter so origin returns dynamic data. | |
| Exploiting exact-match cache rules for web cache deception | Expert | Map the dynamic response onto a cached exact filename (e.g. `/index.html`). | |
