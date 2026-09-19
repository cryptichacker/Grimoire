---
tags: [hacking, web, portswigger, oauth]
type: vulnerability-category
source: https://portswigger.net/web-security/oauth
last-verified: 2026-08-25
---

# OAuth authentication

## Up
- [[PortSwigger]]

## What it is
OAuth 2.0 is an **authorization** framework letting a client app request limited access to a user's data on another service without seeing their credentials. It's widely (mis)used for **authentication** ("Log in with…"). Three parties: **client app**, **resource owner** (user), **OAuth service** (authorization + resource servers).

## Key parameters & flow
- `client_id`, `redirect_uri`, `response_type` (`code` or `token`), `scope`, `state` (CSRF token).
- **Authorization code** grant (secure): browser gets a `code`; the client swaps it server-side (with `client_secret`) for an access token at `/token`.
- **Implicit** grant (weaker): the access token comes straight back in the URL fragment.
- For login, the client reads the user's identity from a `/userinfo` endpoint.
- Config discovery: `/.well-known/oauth-authorization-server`, `/.well-known/openid-configuration`.

## Vulnerabilities
- **Implicit-grant misimplementation** — the client trusts user data POSTed to it after the flow; change the email/id in that POST to log in as anyone.
- **Missing `state` (CSRF)** — with no `state`, an attacker starts a flow and tricks the victim into completing it, linking the victim's account to the attacker's social login (or vice versa).
- **`redirect_uri` weaknesses** — leak the code/token to an attacker server via loose validation: path traversal, extra params, `localhost`-style domains (`localhost.evil.net`), duplicate `redirect_uri`, or chaining an **open redirect** on a whitelisted domain. Tokens/codes also leak through `Referer` (HTML injection `<img src>`) or dangerous client-side JS.
- **Scope upgrade** — add extra `scope` at `/token` (code flow) or `/userinfo` (implicit) if the server doesn't validate against the originally granted scope.
- **Unverified registration** — register at the OAuth provider with the victim's email (unverified), then log in to clients as them.

## Prevention
- Providers: strictly validate `redirect_uri` against a whitelist at **both** authorization and token exchange; enforce scope immutability; verify user email/registration.
- Clients: always send and check `state`; validate token/code responses; never trust unverified provider data.

## Labs
6 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Authentication bypass via OAuth implicit flow | Apprentice | Change the email in the POST that logs you in. | |
| Forced OAuth profile linking | Practitioner | Missing `state`; force-link the victim's account to yours. | |
| OAuth account hijacking via redirect_uri | Practitioner | Steal the code by manipulating `redirect_uri`. | |
| Stealing OAuth access tokens via an open redirect | Practitioner | Chain a whitelisted open redirect to exfil the token. | |
| Stealing OAuth access tokens via a proxy page | Practitioner | Use a gadget page to leak the fragment token. | |
| SSRF via OpenID dynamic client registration | Practitioner | Register a client with a malicious logo/jwks URI for SSRF. | |
