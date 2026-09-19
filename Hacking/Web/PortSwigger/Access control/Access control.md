---
tags: [hacking, web, portswigger, access-control]
type: vulnerability-category
source: https://portswigger.net/web-security/access-control
last-verified: 2026-08-25
---

# Access control

## Up
- [[PortSwigger]]

## What it is
Access control is the enforcement of restrictions on who can do what. It depends on **authentication** (who you are) and **session management** (linking requests to you). Broken access control is common and often critical.

Three types:
- **Vertical** — restricts sensitive functionality to certain user types (e.g. only admins delete accounts).
- **Horizontal** — restricts a resource to specific users (view *your* transactions, not others').
- **Context-dependent** — restricts actions based on application state / correct sequence (e.g. can't change an order after payment).

## Vertical privilege escalation
- **Unprotected functionality** — admin URLs (`/admin`) enforce nothing; found via `robots.txt` or wordlists. **Security by obscurity** (`/admin-panel-yb556`) fails because the URL leaks in JavaScript or elsewhere.
- **Parameter-based** — the app trusts a role value it lets you control (hidden field, cookie, query param):
  ```
  /home.jsp?admin=true
  /home.jsp?role=1
  ```
- **Platform-layer bypass** — front-end restricts a path but the app honours override headers:
  ```
  X-Original-URL: /admin/deleteUser
  X-Rewrite-URL: /admin/deleteUser
  ```
  send `GET /` with the header to reach the "blocked" path.
- **Method-based bypass** — controls enforced only on `POST` but the action also works via `GET` (or vice versa).
- **URL-matching discrepancies** — case (`/ADMIN/...`), Spring suffix matching (`/admin/deleteUser.anything`), or trailing slash (`/admin/deleteUser/`) evading a rule.

## Horizontal privilege escalation & IDOR
Change an identifier to access another user's resource:
```
/myaccount?id=123   →   /myaccount?id=124
```
An **IDOR** is exactly this: user input used to reference an object directly, with no ownership check.
- **Predictable IDs** (incrementing integers) enumerate easily.
- **GUIDs** are harder but often **leak elsewhere** (messages, reviews, logs).
- A **redirect** on unauthorized access still counts if the response body already contains the victim's data.

## Horizontal-to-vertical escalation
Use a horizontal flaw to reach a *privileged* user's account (e.g. `id=administrator`), then read disclosed credentials or use their change-password function to gain admin.

## Multi-step process flaws
An action split across steps may protect steps 1–2 but not the final confirm step; submit the last step directly with crafted parameters.

## Referer-based access control
If a sub-page (`/admin/deleteUser`) only checks the `Referer` header points at `/admin`, forge that header — it's fully attacker-controlled.

## Location-based
Geo restrictions (via IP/geolocation) are bypassable with VPNs/proxies or by manipulating client-side geolocation.

## Prevention
- Never rely on obfuscation alone.
- **Deny by default** for anything not meant to be public.
- Use a **single, application-wide** enforcement mechanism.
- Require developers to declare allowed access per resource.
- Thoroughly audit and test access controls.

## Labs
13 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Unprotected admin functionality | Apprentice | Find `/administrator-panel` via `robots.txt`, use it. | |
| Unprotected admin functionality with unpredictable URL | Apprentice | Recover the admin URL leaked in client-side JS. | |
| User role controlled by request parameter | Apprentice | Set `Admin=true` cookie / param. | |
| User role can be modified in user profile | Apprentice | Update `roleid` via the profile-update JSON. | |
| User ID controlled by request parameter | Apprentice | Change `id` to another user to read their data/API key. | |
| User ID controlled by request parameter, with unpredictable user IDs | Apprentice | Harvest the victim's GUID leaked in a post/blog, then use it. | |
| User ID controlled by request parameter with data leakage in redirect | Apprentice | Read the victim's data from the body of a 302 response. | |
| User ID controlled by request parameter with password disclosure | Apprentice | Read the admin's password from their prefilled account page. | |
| Insecure direct object references | Apprentice | Access `/download-transcript/N.txt` for other users. | |
| URL-based access control can be circumvented | Practitioner | `X-Original-URL: /admin` to bypass the front-end block. | |
| Method-based access control can be circumvented | Practitioner | Replay the admin action with a different HTTP method. | |
| Multi-step process with no access control on one step | Practitioner | Submit the final upgrade step directly as a low-priv user. | |
| Referer-based access control | Practitioner | Forge the `Referer` header on the admin sub-action. | |
