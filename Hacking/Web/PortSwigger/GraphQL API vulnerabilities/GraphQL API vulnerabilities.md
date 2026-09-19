---
tags: [hacking, web, portswigger, graphql]
type: vulnerability-category
source: https://portswigger.net/web-security/graphql
last-verified: 2026-08-25
---

# GraphQL API vulnerabilities

## Up
- [[PortSwigger]]

## What it is
GraphQL exposes a **single endpoint** handling **queries** (read) and **mutations** (write), described by a **schema** and often explorable via **introspection**. Vulnerabilities come from missing access control on arguments, over-permissive introspection, and abuse of aliases/batching.

## Finding the endpoint
- Common paths: `/graphql`, `/api`, `/api/graphql`, `/graphql/api`, `/v1/graphql`.
- **Universal query** to confirm: `query{__typename}` returns `{"data":{"__typename":"query"}}`.
- Try alternative methods (`GET`, or `POST` with `application/x-www-form-urlencoded`) — some are less protected.

## Exploiting unsanitized arguments (IDOR-like)
If arguments aren't authorized, request objects you shouldn't see (e.g. an unlisted product):
```graphql
query { product(id: 3) { id name listed } }
```
Enumerate IDs to pull hidden/other users' data.

## Introspection
Check it's enabled: `{"query":"{__schema{queryType{name}}}"}`. A **full introspection query** dumps every type, query, mutation, and field — the whole schema.

## Bypassing defenses
- **Introspection filters** — a naive regex block on `__schema` can be defeated by inserting a newline/space after it:
  ```
  query{__schema
  {queryType{name}}}
  ```
  Or run introspection over a method (GET) where it wasn't disabled.
- **Rate limiting / brute force** — use **aliases** to pack many operations into one HTTP request:
  ```graphql
  query { a: checkCode(code:"1"){valid} b: checkCode(code:"2"){valid} ... }
  ```
- **Batching** — send an array of queries, or aliased calls, to brute-force codes/credentials in a single request.

## CSRF over GraphQL
If the endpoint accepts non-JSON content types (e.g. `x-www-form-urlencoded` or GET) and has no CSRF token, a malicious page can forge state-changing mutations.

## Prevention
- Disable introspection on **private** APIs; audit exposed fields on public ones.
- Enforce object/field-level **authorization** on every resolver.
- Limit query depth, alias/field counts, and byte size; add cost analysis to stop batching abuse.
- Accept only **JSON POST** and require CSRF tokens / validate content type.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Accessing private GraphQL posts | Apprentice | Query an unlisted post by ID directly. | |
| Accidental exposure of private GraphQL fields | Apprentice | Use introspection to find and query a hidden field (credentials). | |
| Finding a hidden GraphQL endpoint | Practitioner | Locate the endpoint (universal query) with introspection re-enabled via a trick. | |
| Bypassing GraphQL brute force protections | Practitioner | Alias many login attempts into one request. | |
| Performing CSRF exploits over GraphQL | Practitioner | Send a form-encoded mutation from an attacker page. | |
