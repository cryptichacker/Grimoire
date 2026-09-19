---
tags: [hacking, web, portswigger, web-cache-poisoning]
type: vulnerability-category
source: https://portswigger.net/web-security/web-cache-poisoning
last-verified: 2026-08-25
---

# Web cache poisoning

## Up
- [[PortSwigger]]

## What it is
Web cache poisoning makes the cache store a **harmful response** and serve it to other users at scale. The attacker gets the back-end to produce a malicious response, then gets it cached under a normal cache key. Contrast with **cache deception**, which stores a *victim's sensitive* response for the attacker to read.

## How caches work
The cache decides "same request?" using a **cache key** — a subset of the request (usually method + URL/path + Host). Components *in* the key are **keyed**; components ignored by the key but still processed by the back-end are **unkeyed**. Poisoning abuses **unkeyed inputs**: they don't change which cache entry is hit, but they do change the response that gets stored.

## Methodology
1. **Identify unkeyed inputs** — add random headers/params and watch for reflected/altered responses; automate with **Param Miner**. Always add a unique **cache buster** so you don't poison real users while testing.
2. **Elicit a harmful response** — find an unkeyed input reflected unsafely (XSS, open redirect) or that changes generated content (e.g. a header used to build a script/link URL).
3. **Get it cached** — ensure the malicious response is cacheable (right path/extension/status) and lands under the target key.

## Exploiting cache design
- **Unkeyed headers** — `X-Forwarded-Host`, `X-Forwarded-Scheme`, `X-Host` reflected into a link/script/redirect → cached XSS or redirect to an attacker domain.
- **Unkeyed cookie** — a cookie reflected into the response poisons everyone who hits that entry.
- **Multiple headers** — combine several unkeyed headers (e.g. host + scheme) to build a working exploit.
- **DOM-based** — poison an unkeyed value consumed by client-side script for XSS.
- **Exposing too much** — responses that reflect request data broadly widen the surface.

## Exploiting cache-key flaws
- **Unkeyed port / query string / parameter** — inputs excluded from the key let you inject while still hitting the shared entry.
- **Cache parameter cloaking** — hide an extra parameter using a delimiter the cache and origin parse differently.
- **Normalized cache keys** — cache and origin normalize (decode) the URL differently, so an encoded payload is stored under the clean key.
- **Fat GET** — a GET with a body the origin honours but the cache ignores.
- **Cache key injection** — manipulate how the key itself is built.

## Prevention
- Don't cache dynamic content; cache only truly static responses and mark others `Cache-Control: no-store`.
- Strip unnecessary/unkeyed headers before back-end processing (or key on them).
- Disable "fat GET" support; normalize consistently between cache and origin.
- Patch client-side sinks even if they look unreachable; vet third-party cache/CDN defaults.

## Labs
13 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Web cache poisoning with an unkeyed header | Practitioner | Reflect `X-Forwarded-Host` into an imported script → cached XSS. | |
| Web cache poisoning with an unkeyed cookie | Practitioner | Poison via a reflected cookie value. | |
| Web cache poisoning with multiple headers | Practitioner | Combine host + scheme headers to build the exploit. | |
| Targeted web cache poisoning using an unknown header | Practitioner | Find the secret unkeyed header (Param Miner), target a victim. | |
| Web cache poisoning via an unkeyed query string | Practitioner | Query string unkeyed; inject a reflected XSS. | |
| Web cache poisoning via an unkeyed query parameter | Practitioner | A specific parameter is excluded from the key. | |
| Parameter cloaking | Practitioner | Hide a param behind a delimiter the cache/origin parse differently. | |
| Web cache poisoning via a fat GET request | Practitioner | Send a GET body the origin uses but the cache ignores. | |
| URL normalization | Practitioner | Encoded payload normalized differently by cache vs origin. | |
| Cache key injection | Expert | Manipulate cache-key construction to store the payload. | |
| Internal cache poisoning | Expert | Poison an internal cache layer. | |
| Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria | Expert | Poison an unkeyed value feeding a DOM sink. | |
| Combining web cache poisoning vulnerabilities | Expert | Chain several unkeyed inputs. | |
