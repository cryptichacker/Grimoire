---
tags: [hacking, web, portswigger, csrf]
type: vulnerability-category
source: https://portswigger.net/web-security/csrf
last-verified: 2026-08-25
---

# Cross-site request forgery (CSRF)

## Up
- [[PortSwigger]]

## What it is
CSRF induces a logged-in victim's browser to send a state-changing request the victim didn't intend — because the browser automatically attaches their session cookie. The attacker triggers an action (change email, transfer funds) as the victim.

## Conditions required
1. **A relevant action** worth performing (privilege/data change).
2. **Cookie-based session handling** — the request is authenticated solely by an automatically-sent cookie.
3. **No unpredictable parameters** — every value in the request is known/guessable by the attacker.

## Impact
Account takeover (change email/password), financial actions, or any privileged change — full account compromise if the victim is privileged.

## Constructing & delivering
Host an auto-submitting form on an attacker page:
```html
<form action="https://vulnerable-website.com/email/change" method="POST">
  <input type="hidden" name="email" value="pwned@evil-user.net">
</form>
<script>document.forms[0].submit();</script>
```
Deliver via a link/iframe on a page the victim visits (email, ad, comment). GET-based actions can be a bare `<img>`.

## Defenses & bypasses

### CSRF tokens
Server issues an unpredictable, per-session token the client must echo on sensitive requests. Common flaws:
- **Validation depends on method** — enforced on POST but not GET; resend as GET.
- **Validation only if token present** — omit the token parameter entirely.
- **Token not tied to the session** — use a valid token from your own account.
- **Token tied to a non-session cookie** — set/inject that cookie via a separate vector.
- **Token duplicated in a cookie (double-submit)** — set both to the same attacker-chosen value.

### SameSite cookies
- **Strict** — never sent cross-site. **Lax** (common default) — sent only on top-level GET navigations. **None** — sent cross-site (requires `Secure`).
- Bypasses: use a **GET**/method-override under Lax; find an **on-site gadget** (client-side redirect / open redirect) so the request originates same-site; abuse a **sibling domain** (e.g. via XSS) that shares the cookie; exploit **cookie refresh** timing windows in Lax.

### Referer-based validation
- If validation is **skipped when `Referer` is absent**, suppress it (`<meta name="referrer" content="no-referrer">`).
- If it only **checks the domain appears** in the Referer, put the expected domain in your URL/query (`https://evil.com/?vulnerable-website.com`).

## Prevention
- Use **cryptographically strong, per-session CSRF tokens**, tied to the session, in a hidden field or custom header, validated on every state-changing request.
- Set session cookies **`SameSite=Strict`** (or Lax) as defense-in-depth.
- Don't rely on Referer alone.

## Labs
12 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| CSRF vulnerability with no defenses | Apprentice | Auto-submitting form changes the victim's email. | |
| CSRF where token validation depends on request method | Practitioner | Resend the request as GET without a token. | |
| CSRF where token validation depends on token being present | Practitioner | Omit the token parameter. | |
| CSRF where token is not tied to user session | Practitioner | Use a token generated for your own account. | |
| CSRF where token is tied to non-session cookie | Practitioner | Inject your token's matching cookie via a header/gadget. | |
| CSRF where token is duplicated in cookie | Practitioner | Set both cookie and body to the same value (double-submit). | |
| SameSite Lax bypass via method override | Practitioner | Turn POST into GET with `_method`/override to send under Lax. | |
| SameSite Strict bypass via client-side redirect | Practitioner | Bounce through an on-site client-side redirect so it's same-site. | |
| SameSite Strict bypass via sibling domain | Practitioner | Use XSS on a sibling domain sharing the cookie. | |
| SameSite Lax bypass via cookie refresh | Practitioner | Trigger a cookie refresh (e.g. via OAuth) inside the 2-min window. | |
| CSRF where Referer validation depends on header being present | Practitioner | Suppress the `Referer` with a no-referrer policy. | |
| CSRF with broken Referer validation | Practitioner | Include the expected domain in your URL so the substring check passes. | |
