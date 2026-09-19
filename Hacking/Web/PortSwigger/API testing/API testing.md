---
tags: [hacking, web, portswigger, api-testing]
type: vulnerability-category
source: https://portswigger.net/web-security/api-testing
last-verified: 2026-08-25
---

# API testing

## Up
- [[PortSwigger]]

## What it is
APIs (typically REST/JSON) are a large, often under-tested attack surface. Testing them means discovering endpoints and their behaviour, then probing for missing access control, hidden parameters, mass assignment, and server-side parameter pollution.

## Recon
- Look for API documentation: `/api`, `/swagger/index.html`, `/openapi.json`, `/api-docs`.
- Walk base paths: from `/api/v1/users/123`, also try `/api/v1/users`, `/api/v1`, `/api`.
- Extract endpoints from client-side JS (JS Link Finder), and crawl/audit OpenAPI/JSON/YAML docs with Burp Scanner.

## Interacting with endpoints
- Test different **HTTP methods** on the same endpoint (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`) — `OPTIONS` reveals allowed methods.
- Change the **`Content-Type`** (e.g. JSON ↔ XML) to trigger errors, expose different parsers, or bypass controls.

## Hidden parameters & mass assignment
- Mine for hidden parameters with Burp Intruder / Param Miner.
- **Mass assignment (auto-binding)** — frameworks bind request fields straight onto internal objects. If a GET response reveals a field like `"isAdmin": false`, try sending it:
  ```json
  { "username":"wiener", "email":"wiener@example.com", "isAdmin": true }
  ```
  If accepted without checks, you escalate privileges.

## Server-side parameter pollution
The app embeds your input into a *server-side* request to another API. Inject extra parameters/delimiters:
- **Query string** — inject `&`, `#`, `%23`, override or truncate parameters (e.g. `field=x%26extra=y`).
- **REST paths** — inject path traversal/segments to reach a different internal route.
- **Structured formats** — inject JSON/XML structure that merges into the back-end request.

## Prevention
- Secure and don't over-expose documentation; keep it accurate across all versions.
- Allow-list permitted HTTP methods and validate expected content types.
- For mass assignment, **allow-list** the properties a client may set and block sensitive ones.
- Return generic errors; apply auth/rate limits uniformly across versions.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Exploiting an API endpoint using documentation | Apprentice | Find `/api` docs, use the DELETE/PATCH endpoint to modify another user. | |
| Exploiting server-side parameter pollution in a query string | Practitioner | Inject `%26`/`%23` into a field to add/override a back-end parameter (reset token). | |
| Finding and exploiting an unused API endpoint | Practitioner | Discover a hidden method/endpoint (via `OPTIONS`) and change the price. | |
| Exploiting a mass assignment vulnerability | Practitioner | Add a hidden field (e.g. discount/`isAdmin`) to the order/user JSON. | |
| Exploiting server-side parameter pollution in a REST URL | Expert | Inject path segments/encoded delimiters to hit a different internal route. | |
