---
tags: [hacking, web, portswigger, request-smuggling]
type: vulnerability-category
source: https://portswigger.net/web-security/request-smuggling
last-verified: 2026-08-25
---

# HTTP request smuggling

## Up
- [[PortSwigger]]

## What it is
Request smuggling interferes with how a chain of servers (front-end proxy → back-end) parses a sequence of HTTP requests on a shared connection. When the two ends **disagree on where one request ends and the next begins**, an attacker prepends part of a request that the back-end treats as the start of the *next* user's request — bypassing controls, poisoning caches, and capturing others' requests. It's an HTTP/1 problem (and HTTP/2-downgrade problem).

## Root cause: CL vs TE
HTTP/1 has two ways to state body length — `Content-Length` (CL) and `Transfer-Encoding: chunked` (TE). If both are present the spec says ignore CL, but real servers disagree, especially when TE is **obfuscated**.

**CL.TE** — front-end uses CL, back-end uses TE:
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```
Back-end stops at the `0` chunk; `SMUGGLED` begins the next request.

**TE.CL** — front-end uses TE, back-end uses CL:
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0

```
Back-end reads only 3 bytes; the rest is left for the next request. (In Burp, disable "Update Content-Length".)

**TE.TE** — both support TE, but one is tricked into ignoring an obfuscated header:
```
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding:[tab]chunked
 Transfer-Encoding: chunked
X: X[\n]Transfer-Encoding: chunked
```

## Detecting & confirming
- **Timing** — a CL.TE probe makes the back-end wait for more data (delay); TE.CL likewise hangs.
- **Differential responses** — smuggle a prefix that changes the response to a following request, proving desync.

## Exploiting
- **Bypass front-end controls** — smuggle a request to a path the front-end would block.
- **Reveal front-end rewriting** — capture headers the front-end adds (e.g. client IP, auth).
- **Capture other users' requests** — smuggle a request that stores the next request in a retrievable place.
- **Reflected XSS** — deliver a payload to the next user without needing them to click.
- **Cache poisoning / deception** via smuggled requests.

## HTTP/2
End-to-end HTTP/2 is immune (single length mechanism). But **HTTP/2→HTTP/1 downgrading** reintroduces it: **H2.CL**, **H2.TE**, plus HTTP/2-only vectors — **CRLF injection** in header values, header-name injection, request-line/pseudo-header manipulation, **request tunnelling**, **response queue poisoning**, and **request splitting**. Browser-powered variants (**CL.0**, **client-side desync**, **pause-based desync**) reach sites with no proxy access and poison browser caches.

## Prevention
- Use **end-to-end HTTP/2**; don't downgrade.
- Make the front-end **normalize** ambiguous requests and the back-end **reject** them (and close the connection).
- Never assume a request has no body (defeats CL.0); reject headers with newlines, colons in names, or spaces in the method.

## Labs
~22 labs (large topic). Representative set — verify titles/count against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Basic CL.TE vulnerability | Practitioner | Smuggle `SMUGGLED` past a CL front-end / TE back-end. | |
| Basic TE.CL vulnerability | Practitioner | TE front-end / CL back-end; disable auto Content-Length. | |
| Obfuscating the TE header | Practitioner | Use a TE.TE obfuscation one side ignores. | |
| Confirming a CL.TE vulnerability via differential responses | Practitioner | Smuggle a request to a 404 path; next request 404s. | |
| Confirming a TE.CL vulnerability via differential responses | Practitioner | As above for TE.CL. | |
| Bypass front-end security controls, CL.TE | Practitioner | Smuggle a request to `/admin`. | |
| Bypass front-end security controls, TE.CL | Practitioner | Same via TE.CL. | |
| Reveal front-end request rewriting | Practitioner | Capture the header the front-end adds. | |
| Capture other users' requests | Practitioner | Smuggle a comment-post that stores the next request. | |
| Deliver reflected XSS | Practitioner | Smuggle a `User-Agent` XSS to the next user. | |
| Perform web cache poisoning | Practitioner | Smuggle a redirect that gets cached. | |
| Perform web cache deception | Practitioner | Smuggle to cache another user's data. | |
| H2.CL request smuggling | Expert | Downgrade attack: HTTP/2 front-end, CL back-end. | |
| Response queue poisoning via H2.TE | Expert | Desync the response queue to steal responses. | |
| HTTP/2 request smuggling via CRLF injection | Expert | Inject CRLF into an HTTP/2 header value. | |
| HTTP/2 request splitting via CRLF injection | Expert | Split one H2 request into two H1. | |
| Bypassing access controls via HTTP/2 request tunnelling | Expert | Tunnel to reach restricted content. | |
| Web cache poisoning via HTTP/2 request tunnelling | Expert | Tunnel + cache poison. | |
| Client-side desync | Expert | Browser-driven desync, no proxy access. | |
| Browser cache poisoning via client-side desync | Expert | Poison the victim's browser cache. | |
| Server-side pause-based request smuggling | Expert | Exploit a pause in back-end reads. | |
| CL.0 request smuggling | Expert | Back-end ignores the body entirely. | |
