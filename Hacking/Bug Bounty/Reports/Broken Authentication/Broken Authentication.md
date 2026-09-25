---
tags: [hacking, bug-bounty, authentication, session, ato, reports]
type: log
---

# Broken Authentication

## Up
- [[Reports]]

Disclosed **broken authentication & session management** reports — account takeover via password-reset flaws, OAuth misuse, 2FA bypass, weak/identifiable tokens, session fixation. See [[Pattern]] → Broken Authentication.

## Reports

### 2026-09-25 — Taskcluster web-server OAuth2 authorization codes are reusable and the exchange handler checks the wrong expiry (Mozilla) — n/a
- Source: [HackerOne #3734676](https://hackerone.com/reports/3734676)
- Type: Broken authentication — OAuth2 authorization-code replay
- Summary: In `services/web-server/src/servers/oauth2.js` the token-exchange handler neither consumed the authorization code nor enforced the code's own expiry, so a leaked code could be redeemed repeatedly — and long past the ten-minute lifetime RFC 6749 requires — to mint fresh bridge access tokens for the original user.
- Technique / pattern: Two checks to run against any OAuth2 or magic-link exchange: redeem the same code twice and see whether the second attempt still returns a token, then redeem it again after its nominal lifetime. Also check *which* timestamp the handler compares against — an expiry check reading the session's or token's field instead of the code's passes while enforcing nothing.
- Takeaway: Authorization codes, password-reset tokens and magic links must be single-use and expiry-checked against their own issue time. Delete or mark the code inside the same transaction that issues the token, so a replay cannot race the invalidation.

### 2026-09-25 — CVE-2026-13608: OpenLDAP SASL state confusion ends negotiation without authenticating (curl) — n/a
- Source: [HackerOne #3822248](https://hackerone.com/reports/3822248)
- Type: Broken authentication — protocol state-machine confusion
- Summary: In `lib/openldap.c`, `oldap_state_sasl_resp()` advanced to `OLDAP_STOP` whenever `Curl_sasl_continue()` returned `CURLE_OK` with `progress != SASL_INPROGRESS`, without distinguishing "authentication completed successfully" from "idle, no mechanism available". A malicious LDAP server could send a malformed SASL challenge to push the connection past negotiation.
- Technique / pattern: Audit authentication state machines for transitions where two different conditions collapse into the same next state — particularly where a success path and an "unable to proceed" path share a return code. In client-side protocol code, the malicious-server direction is the one that gets least attention.
- Takeaway: A state machine must treat "authenticated" and "nothing left to try" as distinct outcomes; when a negotiation ends without a positive proof of authentication, the connection has to fail closed rather than continue.

### 2026-09-25 — Session hijacking via reusable JSESSIONID exposes OAuth token and profile data (NASA Vulnerability Disclosure Program) — n/a (P4)
- Source: [Bugcrowd 553cc72a](https://bugcrowd.com/disclosures/553cc72a-72b5-4217-8b72-7c4a32ceffed/critical-session-hijacking-via-reusable-jsessionid-exposes-oauth-token-and-profile-data)
- Type: Broken session management — session identifier remains valid after it should be retired
- Summary: A `JSESSIONID` cookie stayed usable after the point where it should have been invalidated, so replaying a captured value re-established an authenticated session and returned the victim's OAuth token and profile information.
- Technique / pattern: The standard session-lifecycle matrix: record the session cookie, then log out, change the password, and let the session idle out, replaying the old value after each event. Note also what the re-established session hands back — a response containing an OAuth access token turns one stolen cookie into durable API access.
- Takeaway: Server-side session state must be destroyed on logout, credential change and expiry — clearing the cookie client-side proves nothing. Never echo long-lived tokens into responses that a replayed session can reach.

### 2026-09-25 — One-click board takeover and DELETE CSRF via path traversal in signature verification (Trello) — 20 points (P2)
- Source: [Bugcrowd 4b84e39f](https://bugcrowd.com/disclosures/4b84e39f-82fc-471a-9595-d27b347b3210/one-click-takeover-of-the-victim-s-board-and-a-delete-csrf-that-can-permanently-delete-any-workspace-or-board-the-victim-has-access-to)
- Type: Broken authentication / CSRF via path traversal in a verification parameter
- Summary: The signature-verification step for board access built a request from a path component that accepted `../` sequences, so a crafted link pointed the verifier at an arbitrary endpoint; any `200` response satisfied the check and granted board access. The same traversal primitive re-routed `DELETE` requests to organization-deletion endpoints, letting a single clicked link destroy workspaces and boards.
- Technique / pattern: When a server validates a request by calling a URL assembled from user input, traversal in that input re-points the call at an endpoint the attacker chooses — and a check that only tests for an HTTP status is satisfied by any reachable route. The researcher then scripted the post-verification actions to copy board contents before the victim noticed.
- Takeaway: Verification must compare a cryptographic signature over canonicalized input, never "did fetching this constructed URL return 200". Normalize and reject traversal in every path segment, and treat a verifier that follows attacker-influenced URLs as an authorization bypass primitive.

### 2026-09-24 — Authentication bypass through HTTP request smuggling on apm.ap.tesla.services (Tesla) — 40 points (P3)
- Source: [Bugcrowd 5e1f7404](https://bugcrowd.com/disclosures/5e1f7404-5421-4f3d-916a-443446afbb52/authentication-bypass-through-http-request-smuggling-on-https-apm-ap-tesla-services)
- Type: HTTP request smuggling (CL.TE) — front-end authentication bypassed
- Summary: The front end and back end disagreed about request length when both `Content-Length` and `Transfer-Encoding: chunked` were present, letting a second request be smuggled past the proxy that enforced authentication. The normally `401` endpoint `/metrics` returned its contents.
- Technique / pattern: Where authentication lives in a reverse proxy rather than in the application, a desync becomes an authorization bypass for every route behind it. Confirm the desync with timing behaviour first, then prove impact by smuggling a request to a known-protected path and watching the status change from `401` to `200`.
- Takeaway: An edge-enforced auth control is only as sound as the two servers' agreement about where one request ends; the fix is consistent parsing, not another rule at the edge.

### 2026-09-24 — Session not invalidated on password change on my.zapinfo.io (Indeed) — $100 (P4)
- Source: [Bugcrowd cbd696e7](https://bugcrowd.com/disclosures/cbd696e7-85cc-45d4-932c-089d0a276a02/session-is-not-invalidated-on-password-change-https-my-zapinfo-io)
- Type: Broken session management — no server-side revocation after a credential change
- Summary: On the ZapInfo platform, which carried its own credential store alongside Indeed credentials, existing sessions stayed valid after a password change, so an attacker holding a stolen session kept access straight through the victim's remediation.
- Technique / pattern: Sign in from two browsers, change the password in one, and keep issuing authenticated requests from the other. Test this separately for every credential path the application supports — a product that added local credentials beside an SSO provider often wires revocation into only one of them.
- Takeaway: Changing a password is the user's remediation step; if it does not revoke sessions server-side it is only a UI gesture, and which login path was used decides whether the revocation code runs at all.

### 2026-09-24 — Internal admin panel reached by bypassing OAuth and generating a valid session (Mapbox) — n/a
- Source: [HackerOne #294911](https://hackerone.com/reports/294911)
- Type: Authentication bypass — session established without a completed OAuth verification
- Summary: Mapbox's internal portal created its application session in a way that did not depend on the OAuth verification having succeeded, so a valid session could be generated directly and used to read data behind the admin interface. Mapbox fixed it by reworking session handling.
- Technique / pattern: Treat the identity-provider handshake and the local session as two separate things. After the redirect back from the provider, replay or tamper with the callback and check whether the application still issues a session — the flaw is almost always in the step that converts the provider's answer into a local cookie.
- Takeaway: OAuth proves who the user is; it does not by itself decide who gets a session. Bind session creation to a server-side validated assertion and to the authorization state that started the flow.

### 2026-09-24 — Authentication bypass via improper input validation in the login-token method (Rocket.Chat) — n/a
- Source: [HackerOne #1447619](https://hackerone.com/reports/1447619)
- Type: Authentication bypass — unvalidated request data forwarded to a login handler
- Summary: Data from HTTP POST requests was forwarded to Rocket.Chat's hardcoded login handlers, including the `login-token` method, without adequate validation of the supplied token, opening a path to authenticate as another user.
- Technique / pattern: Applications built on Meteor-style method dispatch expose every registered login handler to whatever the HTTP layer forwards. Enumerate the handlers the framework registers and send each a type-confused argument — an object or array where a string token is expected — since handlers commonly validate the value but not its type.
- Takeaway: Every login handler a framework registers is an authentication entry point; validate the type and shape of credentials at the HTTP boundary, before dispatch picks a handler.

### 2026-09-24 — Authentication bypass: session created for NASA SIPS administrator account with any password (NASA VDP) — n/a (P2)
- Source: [Bugcrowd ac8d1f0e](https://bugcrowd.com/disclosures/ac8d1f0e-c0cf-4784-97b2-ba10e7b49da4/authentication-bypass-session-created-for-nasa-sips-administrator-account-with-any-password)
- Type: Broken authentication — login endpoint issues a session despite failed password validation
- Summary: A NASA SIPS backend API login endpoint contained improper password-validation logic: submitting an administrative username with an arbitrary password still resulted in a session being created, giving unauthorized access to the administrative account. Triaged P2 and remediated; no sensitive data was exfiltrated during discovery.
- Technique / pattern: Compare the *whole* response to a wrong-password attempt against one for a known-bad username, rather than reading the status line. The tells are a `Set-Cookie` or token issued alongside an apparent failure, a body that says "invalid" while the session behind it is already valid, or a front end that blocks on a flag the backend never enforces. Then replay the cookie you were handed against a protected route — that, and not the login response, is the proof.
- Takeaway: A login endpoint has two outcomes that must never drift apart: the message and the session. Always replay the cookie from a "failed" login against an authenticated route before accepting that the login actually failed.

### 2026-09-24 — SAML authentication bypass leading to unauthenticated admin takeover on scijinks.gov / nesdis.noaa.gov (NASA VDP) — n/a (P1)
- Source: [Bugcrowd abfa03c7](https://bugcrowd.com/disclosures/abfa03c7-9c7d-46f7-b255-9766199dac4a/saml-authentication-bypass-leading-to-unauthenticated-admin-takeover-on-scijinks-gov-nesdis-noaa-gov)
- Type: Broken authentication — SAML assertion accepted with no signature validation
- Summary: The SAML Assertion Consumer Service endpoint at `/saml/acs` did not verify that incoming SAML responses were cryptographically signed by the trusted identity provider. A fabricated response carrying no digital signature at all was accepted and processed, so supplying an arbitrary `NameID` yielded a fully authenticated Drupal administrator session in a single unauthenticated HTTP request. Both `scijinks.gov` and `nesdis.noaa.gov` were affected because they share one Drupal instance.
- Technique / pattern: Locate the ACS route (`/saml/acs`, `/saml/consume`, `/sso/acs`, `/simplesaml/module.php/saml/sp/saml2-acs.php`) and `POST` a self-made `SAMLResponse` to it directly instead of driving the IdP login UI. Escalate in order: no signature at all, then a self-signed signature, then signature wrapping of a legitimate assertion — the no-signature case is the fastest and is exactly what failed here. Map which other hostnames share the back end, because an ACS flaw on an obscure domain can own the flagship one.
- Takeaway: The ACS endpoint is unauthenticated by definition, which makes it the highest-value route in any SSO deployment. Test it as a raw `POST`, and always check which hostnames sit on the same CMS instance.

### 2026-09-24 — Authentication bypass in ID4me handling via missing JWT signature verification in User OIDC (Nextcloud) — $2,500
- Source: [HackerOne #3489490](https://hackerone.com/reports/3489490)
- Type: Broken authentication — unverified JWT signature in a federated OIDC flow (CVSS 8.1, High)
- Summary: The Nextcloud User OIDC app, in versions from `0.3.0` through the `6.0.0` line, did not verify the signature on JWTs received through the ID4me identity-discovery flow, so a malicious or attacker-controlled ID4me authority could forge a token and impersonate any user on an affected instance. Fixed in `3.1.0`, `4.1.0`, `5.1.0`, `6.4.0` and `8.3.0`, with disabling ID4me offered as the workaround.
- Technique / pattern: In federated identity the trust question has two halves — is this token signed, and is the signer the issuer we trust for this user. ID4me-style discovery resolves the identity provider dynamically from the identifier the user types, so the second half is attacker-influenced by design and the signature check becomes the only remaining boundary. On open-source targets, read the token-handling path directly and look for a decode call standing where a verify call belongs: `parse` instead of `verify`, `decode(..., verify: false)`, or a JWKS that is fetched but never applied.
- Takeaway: Dynamic issuer discovery combined with unverified signatures is a total authentication bypass. Any code path that turns a user-supplied identifier into an issuer URL needs both signature verification and an issuer allow-list.

### 2026-09-24 — 1-Click Account Takeover via Open Redirect through Regex Bypass in Domain Validation (Khan Academy) — n/a (Critical)
- Source: [HackerOne #3723458](https://hackerone.com/reports/3723458)
- Type: Broken authentication — transfer-token leak via an unescaped dot in a domain allow-list regex
- Summary: Khan Academy's cross-domain login used a `continue` parameter to mint a one-time transfer auth token and pass it in the URL to another Khan Academy subdomain. The `KA_DOMAIN_REGEX` validating that redirect target contained an unescaped dot, so an attacker could register a domain that satisfied the pattern, receive the victim's transfer token, and replay it on a legitimate Khan Academy domain to obtain a full session (`KAAS`, `KAAL`, `KAAC` cookies) as the victim.
- Technique / pattern: Two reusable moves. First, parameters named `continue`, `next`, `returnTo` or `redirect_uri` on an apex domain often trigger a cross-domain authentication handshake rather than a plain redirect — watch for a token appearing in the outbound URL, which is the whole finding. Second, the validating regex is usually shipped to the client: the researcher recovered `KA_DOMAIN_REGEX` from a leaked source map and then read it for the classic flaws — an unescaped `.` matching any character, a missing `$` anchor, a missing `^`.
- Takeaway: A secret travelling through a redirect URL is only as strong as the allow-list validating that URL, and that allow-list is usually a regex you can read out of the bundle or its source map. Validate redirect targets against an exact host allow-list, never a pattern.

### 2026-09-23 — 2FA can be disabled without confirming the account password (Localize (HackerOne)) — bounty awarded (amount undisclosed)
- Source: [HackerOne #783258](https://hackerone.com/reports/783258)
- Type: Broken authentication / missing re-authentication on 2FA disable
- Summary: The `/api/user/two-factor/set` endpoint let a logged-in session turn off (or reconfigure) two-factor authentication without supplying the account password, so anyone holding a session cookie could strip 2FA and reconfigure it.
- Technique / pattern: Observed by capturing the legitimate disable request and noting the server acted on it without any password field — the check for re-authentication on a security-downgrade action was simply absent.
- Takeaway: Disabling or re-enrolling 2FA is a security-downgrade action and must require a fresh password (or step-up) check server-side. A session cookie alone should never be enough to remove a second factor.

### 2026-09-23 — 2FA requirement bypassed via the embedded submission form (HackerOne) — $10,000
- Source: [HackerOne #418767](https://hackerone.com/reports/418767)
- Type: Broken authentication / inconsistent enforcement of a security control
- Summary: A program required reporters to enable 2FA before submitting, but that requirement was enforced only on the standard submission page; the embedded submission form at `hackerone.com/[program]/embedded_submissions/new` performed no equivalent check, so reports could be filed without 2FA. Investigation also surfaced a separate attachment-access issue, raising the total award.
- Technique / pattern: Found by locating an alternate entry point to the same action — the embedded form linked from the program policy page — and confirming the gate present on the primary path was missing there.
- Takeaway: A security control has to be enforced on every path to an action, not just the main UI. Embedded, legacy and API entry points to the same operation are where inconsistent enforcement hides.

### 2026-09-23 — Email address change without verification enabling password-reset takeover (NASA (Bugcrowd)) — n/a (P5, informational)
- Source: [Bugcrowd 2b1f3782](https://bugcrowd.com/disclosures/2b1f3782-127b-4265-acb1-53da1ff66a5c/able-to-change-email-address-without-any-verification-cause-account-takeover)
- Type: Broken authentication / missing verification on sensitive change
- Summary: The account settings flow allowed changing the registered email address with no verification of the new address or re-authentication, which an attacker with a briefly unattended session could chain into a password reset to the new address.
- Technique / pattern: The test is whether a sensitive-field change (email, phone, recovery address) requires re-authentication or a confirmation step; here it required neither, so the new address could immediately receive a reset link.
- Takeaway: Treat email/phone/recovery changes as sensitive actions gated by password re-entry or a confirmation link to the old address. An unverified email change quietly becomes an account-recovery bypass.

### 2026-09-23 — Sessions not invalidated after password reset (Atlassian (Bugcrowd)) — n/a (P4, informational)
- Source: [Bugcrowd 19c9f3c7](https://bugcrowd.com/disclosures/19c9f3c7-e60a-4d29-8cfe-ea2be0b64549/failure-to-invalidate-session-after-password-reset)
- Type: Broken authentication / session management
- Summary: On Atlassian Identity, changing an account password did not terminate other active sessions, so a session established before the reset stayed authenticated afterwards.
- Technique / pattern: Demonstrated by logging into one account in two browsers, changing the password in one, and confirming the second browser remained logged in — the canonical test for session invalidation on credential change.
- Takeaway: A password reset is the primary way a user evicts an attacker, so it must revoke all other sessions server-side. Test that credential changes invalidate concurrent sessions, not just the current one.

### 2026-09-23 — Improper access control in the email/authentication tab (Weblate) — n/a
- Source: [HackerOne #223434](https://hackerone.com/reports/223434)
- Type: Broken authentication / account identity management
- Summary: A flaw in how confirmed and secondary email addresses were added and removed in the authentication tab allowed the account's verified identity set to be manipulated in a way the confirmation flow was meant to prevent.
- Technique / pattern: Walk the full lifecycle of a secondary identity — add, confirm, remove, re-add and re-order — while watching which step actually re-validates ownership; bugs concentrate at removal and re-add, where confirmation is often skipped.
- Takeaway: Email management is part of the authentication surface because the address set drives password recovery — test every transition in that lifecycle, not just the initial confirmation.

### 2026-09-23 — Privilege Escalation via Insufficient Access Controls in Registration (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #796379](https://hackerone.com/reports/796379)
- Type: Broken authentication / privilege escalation at registration
- Summary: The registration endpoint of an education application let a user tamper with the signup request and register directly as an administrator, because the role was taken from client input rather than assigned server-side.
- Technique / pattern: Intercept the registration request and add or alter role, group and permission fields — including ones the form never sends but the back end may accept — then check the privileges of the resulting account.
- Takeaway: Registration is an authentication boundary: any role or entitlement field that reaches the server from the client must be ignored, since mass-assignment at signup grants admin without ever touching a login flow.

### 2026-09-23 — HTTP Desync Attack (Request Smuggling) — Mass Session Hijacking (Foxy.io) — $500 (P1)
- Source: [Bugcrowd #7b175e9d](https://bugcrowd.com/disclosures/7b175e9d-8ff0-47e3-bd33-a8b1e51aa499/http-desync-attack-request-smuggling-mass-session-hijacking)
- Type: HTTP request smuggling leading to session hijacking
- Summary: The front-end server honoured `Content-Length` while the back-end honoured `Transfer-Encoding`, letting an attacker smuggle a request that captured following users' requests — including session cookies and auth tokens — with no user interaction. Disclosed 2022-01-31; traced to a CloudFront desync issue AWS later patched.
- Technique / pattern: Send an ambiguous request carrying both `Content-Length` and `Transfer-Encoding`, time the response to confirm the desync, then park a prefix that appends the next victim's request into an attacker-controlled page where the full headers are logged.
- Takeaway: Wherever a CDN or proxy sits in front of an origin the two may disagree on request boundaries; smuggling turns that disagreement into mass credential capture, so test CDN-fronted hosts specifically.

### 2026-09-23 — Broken Authentication and Session Token Weakness on admin.phacility.com (Phabricator) — n/a
- Source: [HackerOne #1271710](https://hackerone.com/reports/1271710)
- Type: Broken authentication / session management
- Summary: Weak session-token handling in the authentication flow could allow someone other than the account owner to assume a valid session.
- Technique / pattern: Proxy the mobile client's traffic, exercise account operations such as adding an address in the email settings page, and inspect how session tokens are generated, transmitted and re-validated across those requests.
- Takeaway: Proxying the mobile app frequently reveals token handling the web front-end hides, and account-settings flows are the highest-value place to inspect session re-validation.

### 2026-09-22 — Session not invalidated after logout allows session reuse (Genius) — n/a (P5, accepted risk)
- Source: [Bugcrowd #460e8d40](https://bugcrowd.com/disclosures/460e8d40-5bc0-4128-8567-1195a45f550a/session-not-invalidated-after-logout-allowing-session-reuse-server-side-also)
- Type: Session management — no server-side invalidation on logout
- Summary: Session cookies captured before logout stayed valid afterwards; the token was only dropped client-side, and users had to change their password to force invalidation.
- Technique / pattern: Capture an authenticated request, log out in the browser, then replay the captured cookie/token against an authenticated endpoint. Repeat the same test after password change, email change and "log out of all devices" — each is a separate invalidation path.
- Takeaway: Logout must destroy server-side session state, not just clear the cookie; otherwise any token ever leaked (logs, proxies, shared machines) remains usable indefinitely.

### 2026-09-22 — Arbitrary external redirect through SAML RelayState after successful authentication (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #a1363cea](https://bugcrowd.com/disclosures/a1363cea-d7c2-4e36-901d-cc25f0123c26/arbitrary-external-redirect-through-saml-relaystate-after-successful-authentication)
- Type: Broken authentication flow — unvalidated post-login redirect parameter
- Summary: The SAML `RelayState` parameter, which carries the destination a user returns to after the identity provider signs them in, was not validated against an allowlist, so an attacker-crafted login link sent the freshly authenticated user to an external site.
- Technique / pattern: Treat every "where to go after login" carrier — `RelayState`, `redirect_uri`, `next`, `returnTo`, `state` — as an open-redirect candidate and test absolute URLs, protocol-relative `//evil.tld`, and allowlist-normalisation tricks. Post-authentication redirects are the more dangerous half because the victim has just proven they trust the domain.
- Takeaway: Validate redirect targets server-side against a strict allowlist of relative paths or known hosts; an authenticated redirect is a phishing and (when tokens ride in the URL or fragment) a token-theft primitive.

### 2026-09-22 — Account takeover by brute-forcing the password reset token (HackerOne) — n/a
- Source: [HackerOne #17512](https://hackerone.com/reports/17512)
- Type: Broken authentication — no rate limiting on reset-token validation
- Summary: `/users/password/edit?reset_password_token=...` could be attacked directly: with no throttling or lockout, thousands of candidate tokens could be submitted and valid ones identified from the differing response (HTTP 200 versus a 302, and response-length differences), after which the attacker set a new password.
- Technique / pattern: A classic still worth re-testing everywhere — trigger a reset, then measure whether the token endpoint has rate limiting, whether invalid and valid tokens produce distinguishable responses, and how much entropy and lifetime the token really has.
- Takeaway: Reset tokens need high entropy, short expiry, single use *and* rate limiting; and response differences (status, length, timing) are what turns a guessing attack into a practical one.

### 2026-09-22 — Authorization bypass allows changing another user's email address (Revive Adserver) — n/a (CVSS 8.8)
- Source: [HackerOne #3398283](https://hackerone.com/reports/3398283)
- Type: Missing step-up re-authentication on a sensitive account change
- Summary: The UI's "Change Email" feature demanded the current password, but the admin route `POST /admin/agency-user.php` accepted `userid` and `email_address` with no password confirmation, so an authenticated user holding User Access permissions could rewrite an administrator's email and pivot to takeover through password recovery.
- Technique / pattern: Find the second path to a sensitive change — admin panels, bulk edit, API and mobile routes often reimplement the operation without the step-up check the main form enforces. Compare the parameter sets of the two requests: the weaker one usually lacks the credential field entirely.
- Takeaway: Step-up authentication must be enforced in the service layer that performs the change, not on one form; email is an authentication factor, so any unguarded write to it is an account-takeover primitive.
### 2026-09-22 — Group restriction bypass via bearer token in user_oidc (Nextcloud) — $150
- Source: [HackerOne #3572848](https://hackerone.com/reports/3572848)
- Type: Inconsistent auth policy enforcement across login flows (OIDC)
- Summary: The `user_oidc` app enforced the group login restriction (`SETTING_RESTRICT_LOGIN_TO_GROUPS`) in the browser OIDC flow but not during bearer-token validation in `Backend::getCurrentUserId`, so users outside allowed groups could still use the API.
- Technique / pattern: Compared the interactive login path against the API bearer-token path and found the group check only in the first; exploitation needed the OIDC `client_secret` to exchange a code for a bearer token.
- Takeaway: When an app has several ways to authenticate (SSO, bearer tokens, app passwords), verify every login policy is enforced on each path; centralise the check.

### 2026-09-22 — Mass account takeover via unvalidated reset token on email-change endpoint (Stripe (TaxJar)) — n/a
- Source: [HackerOne #1685970](https://hackerone.com/reports/1685970)
- Type: Broken account-recovery flow / auth bypass via alternate path
- Summary: The email-change flow at `POST /accounts/<ACCOUNT_NUMBER>` trusted the account number in the URL instead of binding the action to the reset token's owner, so any account's email could be changed to the attacker's.
- Technique / pattern: Observed the target account was chosen by a URL parameter rather than the token, so one attacker token worked against every account; iterating account numbers enabled bulk takeover with no victim interaction.
- Takeaway: Bind recovery and account-modification actions to the token's own subject, never to a separate client-supplied ID.

### 2026-09-22 — No rate limit in two-factor authentication leads to brute-force bypass (US EPA VDP) — n/a
- Source: [Bugcrowd #6cf5fccf](https://bugcrowd.com/disclosures/6cf5fccf-1018-459a-b9f8-02d18b7a4ce1/no-rate-limit-in-two-factor-authentication-leads-to-bypass-using-bruteforce-attack)
- Type: Missing rate limit on 2FA (P4)
- Summary: The 2FA verification step had no rate limiting, lockout or backoff, so the short numeric code could be brute-forced.
- Technique / pattern: Repeatedly submitted candidate codes to the verification endpoint and observed no throttling, showing the code space could be exhausted.
- Takeaway: A low-entropy code with unlimited guesses is not a second factor; require attempt limits, lockout, and short code lifetimes.

### 2026-09-22 — Account takeover at git.smce.nasa.gov via CVE-2023-7028 (NASA VDP) — n/a
- Source: [Bugcrowd #8414f6bd](https://bugcrowd.com/disclosures/8414f6bd-5eda-463a-86b3-7b855f749a42/account-take-over-at-https-git-smce-nasa-gov)
- Type: Account takeover via known auth CVE (P1)
- Summary: A self-hosted GitLab instance was vulnerable to account takeover via the known CVE-2023-7028 (the disclosure calls it an OAuth misconfiguration); NASA fixed it.
- Technique / pattern: Patch-gap hunting: fingerprint self-hosted products and their versions, then test them against publicly known critical authentication CVEs (CVE-2023-7028 is GitLab's password-reset-to-unverified-email flaw).
- Takeaway: Self-hosted dev platforms often lag on patches; version-fingerprint and check critical auth CVEs early in recon.

### 2026-09-21 — Unauthenticated create/read/delete of any user's data + email relay (NASA JPL Hurricane Watch)
- Source: [Bugcrowd #eb823c31](https://bugcrowd.com/disclosures/eb823c31-5e78-4bf9-9b2a-4c339918e879/unauthenticated-create-read-and-delete-of-any-user-s-data-email-relay-on-jpl-hurricane-watch)
- Type: Broken authentication (missing auth on state-changing API)
- Summary: The Hurricane Watch API required no session, cookie, or token: any actor could create, read, and delete any user's records using only a name identifier, and abuse an unprotected email-relay endpoint.
- Technique / pattern: Call the API's CRUD and email endpoints directly with no credentials, supplying a target's name as the only identifier; every operation succeeds because authentication is simply absent.
- Takeaway: Authentication must gate every state-changing endpoint by default — a public read is one thing, but unauthenticated write/delete and mail relay is total compromise.

### 2026-09-21 — Auth bypass via path-normalization (double slash) on NASA MODAPS OKAPI (NASA VDP)
- Source: [Bugcrowd #c6b4ca39](https://bugcrowd.com/disclosures/c6b4ca39-0432-4180-995d-93ddec8ff614/critical-authentication-bypass-via-path-normalization-double-slash-on-live-nasa-modaps-okapi-production-instance)
- Type: Broken authentication (WAF/backend path-parsing differential)
- Summary: The edge protection layer (AWS ALB) and the backend ASGI server parsed request paths differently, so requests to admin routes using duplicate slashes bypassed the WAF and reached protected logic unauthenticated.
- Technique / pattern: Prefix restricted routes with a duplicate or encoded slash — `//-/admin/*` or `/%2f-/admin/*`; the WAF rule doesn't match the mangled path but the backend normalizes it and serves the admin endpoint.
- Takeaway: Access control enforced only at the edge is defeated by parser disagreements — normalize paths identically front-to-back and enforce authorization at the application, not the proxy.

### 2026-09-21 — Session not invalidated → cookie reuse / 2FA bypass (HackerOne)
- Source: [HackerOne #2469706](https://hackerone.com/reports/2469706)
- Type: Broken authentication (insufficient session expiration)
- Summary: Previously authenticated sessions stayed valid after logout / password change, so a captured session cookie kept granting access — and could sidestep 2FA by relying on the still-valid older token.
- Technique / pattern: Capture a victim's session cookie, then keep using it after the victim logs out or re-authenticates; the old token is never revoked, so it continues to authenticate the attacker.
- Takeaway: Logout, password change, and 2FA enrollment must invalidate all existing sessions server-side — client-side expiry or new-session issuance alone leaves stolen tokens live.

### 2026-09-21 — Password change endpoint bypass via missing rate limit on old-password field (X / xAI)
- Source: [HackerOne #982293](https://hackerone.com/reports/982293)
- Type: Broken authentication (no rate limiting on re-auth)
- Summary: On the FlightSchool platform, the password-change endpoint's current-password check had no rate limiting, so an attacker with a hijacked session could brute-force the old password and complete a takeover.
- Technique / pattern: With an active session, submit a new password and brute-force the intercepted `old_password` field; absent rate limiting, unlimited attempts eventually match and the change succeeds.
- Takeaway: Re-authentication controls (current-password, step-up) are security theater without rate limiting and lockout — throttle the verification field, not just the login form.


### 2026-09-21 — Password-reset token not invalidated after email change (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 762dfdd3](https://bugcrowd.com/disclosures/762dfdd3-80b8-4145-9ea5-d133982ad154/password-reset-link-not-expiring-after-changing-the-email-leads-to-account-takeover)
- Type: Broken Authentication (token lifecycle, P5)
- Summary: A previously issued password-reset link stayed valid (up to one day) even after the account's email was changed, leaving a window for takeover if the old link was captured.
- Technique / pattern: Request a reset link, then change the account email; the earlier token is not revoked, so whoever holds it can still complete a password reset on the account.
- Takeaway: Any change to security-relevant state (email, password) must invalidate outstanding reset tokens and active sessions — time-boxing alone is not invalidation.

### 2026-09-21 — Full account takeover via missing CSRF-token validation on accounts.yoyogames.com (Opera) — $400
- Source: [Bugcrowd disclosure c206f5d2](https://bugcrowd.com/disclosures/c206f5d2-e2f5-493b-9cb3-c94a00d8106b/full-account-takeover-due-to-failure-to-validate-csrf-token-on-accounts-yoyogames-com-urgent)
- Type: Broken Authentication (CSRF, P3)
- Summary: CSRF tokens on `accounts.yoyogames.com` were not validated on the `/profile/details` endpoint (and possibly site-wide), allowing forged cross-site requests to change profile details.
- Technique / pattern: Host a page that auto-submits a cross-origin request to `/profile/details` without a valid token; because the token is unchecked and cookies default to `SameSite=Lax`, the change applies to a logged-in victim.
- Takeaway: A CSRF token present in the form but never verified server-side is no defense — validate it on every state-changing endpoint and set `SameSite` explicitly.

### 2026-09-21 — OTP bypass via response modification for new users (Indeed) — $250
- Source: [Bugcrowd disclosure 761966c0](https://bugcrowd.com/disclosures/761966c0-600f-4265-8f11-1ba044a0ba75/otp-bypass-through-response-modification-for-new-users)
- Type: Broken Authentication (client-side OTP validation, P3)
- Summary: At `indeedchat.indeed.com` the OTP check for new users could be bypassed by editing the server response to the `verifyOtp` request in an intercepting proxy.
- Technique / pattern: Enter any OTP, intercept the `verifyOtp` response in Burp, and flip the failure/status field to a success value; the client accepts it and grants access.
- Takeaway: OTP verification must be enforced server-side — if the client trusts a response field to decide success, tampering the response defeats 2FA entirely.

### 2026-09-21 — Removed user regains access via Forgot-Password (Rewards Genius / Tango Card) — 40 pts
- Source: [Bugcrowd disclosure ffe37a89](https://bugcrowd.com/disclosures/ffe37a89-7f51-412a-8071-2ce6b08cba84/authentication-bypass-leads-to-unauthenticated-use-of-money-and-group-all-actions-including-admin-removal)
- Type: Broken Authentication (deprovisioning bypass, P1)
- Summary: A user removed from a Rewards Genius organization could regain access by using the Forgot-Password flow, which bypassed the organization authentication/removal state.
- Technique / pattern: After being removed, trigger the password-reset flow; the reset path re-established a session without re-checking that the account was deprovisioned from the org.
- Takeaway: Deprovisioning must invalidate every re-entry path — password reset, OAuth link, existing sessions — or a removed member walks back in through the reset flow.

### 2026-09-21 — 2FA bypass via account deactivation + password-reset reactivation (HackerOne) — n/a
- Source: [HackerOne #2463279](https://hackerone.com/reports/2463279)
- Type: Broken Auth (2FA bypass via state transition)
- Summary: An attacker with access to the victim's email could deactivate the account, trigger a password reset, and on reactivation the two-factor requirement was dropped — full takeover without the 2FA secret.
- Technique / pattern: Walk the account through deactivate → reset password → reactivate; the 2FA enrollment is not re-enforced across that state transition, so the reactivated session skips the second factor.
- Takeaway: Map every account state (active/deactivated/recovering/reactivated) and re-test 2FA enforcement from each — factor checks are frequently lost across lifecycle transitions.

### 2026-09-21 — Authentication bypass via PreSignedURL with expired OC-Date/OC-Expires (ownCloud) — $2,000
- Source: [HackerOne #2337427](https://hackerone.com/reports/2337427)
- Type: Broken Auth (signature validator fails open)
- Summary: ownCloud Infinite Scale's PreSignedURL validation on /remote.php/dav/files/ returned success without verifying the signature when the OC-Date/OC-Expires values were expired, allowing unauthenticated file downloads.
- Technique / pattern: Craft a request with expired OC-Date and OC-Expires; the validator short-circuits to 'valid' on the expiry branch and skips the cryptographic signature check, so only username+filename are needed to download.
- Takeaway: Test pre-signed/signature validators on their edge branches (expired, malformed, missing) — a check that returns success on a precondition instead of failing closed is an auth bypass.

### 2026-09-21 — Account takeover via password reset after account deletion (Weblate) — n/a
- Source: [HackerOne #230076](https://hackerone.com/reports/230076)
- Type: Broken Auth (reset ignores account existence)
- Summary: The /accounts/reset/ password-reset flow did not validate that the account still existed, so a deleted account could be reclaimed by requesting a reset and setting a new password.
- Technique / pattern: Delete your account, then request a reset for its email, follow the emailed link and set a new password; recovery re-materializes the deleted account for whoever completes the flow.
- Takeaway: Reset/recovery must confirm the target account is in a valid, existing state — flows that don't check for deleted/removed accounts let them be silently recovered.

### 2026-09-21 — Reset any password by brute-forcing an unrate-limited 6-digit code (pixiv) — bounty
- Source: [HackerOne #703972](https://hackerone.com/reports/703972)
- Type: Broken Auth (OTP brute force in reset)
- Summary: pixiv.net/reminder.php used a 6-digit verification code with no rate limiting on the final reset step, so any user's password could be reset by brute-forcing the code.
- Technique / pattern: Start a reset for the victim's email, then brute-force the 6-digit confirmation code at the final step since submission attempts are unthrottled, and set a new password.
- Takeaway: Any short numeric secret in a reset/OTP flow needs strict rate limiting and lockout on the *verify* step — a 6-digit space is trivially brute-forced without it.


### 2026-09-21 — SAML signup domain-enforcement bypass gives unauthorized org access (HackerOne / PullRequest) — n/a
- Source: [HackerOne #2101076](https://hackerone.com/reports/2101076)
- Type: SSO/SAML enforcement bypass via input-validation flaw
- Summary: HackerOne's signup failed to fully sanitize the email, letting an attacker register a local-password account on a SAML-enforced domain by appending CRLF (%0d%0a) characters, bypassing the redirect-to-SSO enforcement and reaching SSO-linked resources.
- Technique / pattern: Submit a protected-domain email that normally forces SSO, intercept the POST and append %0d%0a to the email so validation treats it as non-enforced while the account is still created for the protected domain, then log in with a chosen password.
- Takeaway: Normalize and strictly validate email input (reject control/CRLF chars and trailing whitespace) before SSO-domain enforcement, and apply the same canonicalization wherever identity is compared.

### 2026-09-21 — Password reset token leak via Host header on third-party website (Shopify) — n/a
- Source: [HackerOne #1092831](https://hackerone.com/reports/1092831)
- Type: Password reset flaw (Host-header token leakage)
- Summary: Password reset tokens were exposed to third-party domains through Host-header-influenced links, so an attacker controlling that domain could capture the token and reset a victim's password. Disclosed 2022-02-10.
- Technique / pattern: Request a reset, observe the reset token carried to an attacker-influenceable host via Host header manipulation or an open redirect in the reset flow, then capture and replay the token.
- Takeaway: Build reset URLs from a server-side allowlisted canonical hostname, never from the request's Host header, and never let reset tokens transit or leak to third-party domains (guard Referer leakage too).

### 2026-09-21 — OAuth misconfiguration leads to account takeover (Reddit) — n/a
- Source: [HackerOne #1815463](https://hackerone.com/reports/1815463)
- Type: OAuth/SSO misbinding (email-based account linking)
- Summary: A misconfiguration in Reddit's Google OAuth flow on accounts.reddit.com let an attacker register a new account with an email already tied to a victim's Google OAuth login, taking over the victim's email identity. Disclosed 2023-05-18.
- Technique / pattern: Sign in via Google OAuth, log out, then use the registration flow to create an account with the same email; the system failed to reconcile the existing OAuth identity with the new signup.
- Takeaway: Bind federated identity canonically to a single verified account; never let a fresh registration or new login method claim an email already tied to an existing SSO identity without re-verification.

### 2026-09-21 — Authentication bypass when using JWT with public keys (8x8 / Jitsi Meet) — n/a
- Source: [HackerOne #1210502](https://hackerone.com/reports/1210502)
- Type: JWT flaw - algorithm confusion (RS256 -> HS256)
- Summary: Jitsi Meet before 2.0.5963 accepted JWTs whose header specified a symmetric algorithm, letting an attacker forge tokens by using the publicly known RSA public key as the HMAC secret to reach protected conference rooms.
- Technique / pattern: On a server configured for RS256, flip the token's alg to HS256 and sign with HMAC using the server's public key as the secret; a verifier that trusts the header's alg validates it.
- Takeaway: Pin the expected signing algorithm server-side and never derive it from the attacker-controlled JWT header; public keys are public and must never be usable as a shared secret.

### 2026-09-21 — Authentication & Registration Bypass in Newspack Extended Access (Automattic) — n/a
- Source: [HackerOne #2472798](https://hackerone.com/reports/2472798)
- Type: JWT - signature not verified -> auth bypass
- Summary: The Newspack Extended Access plugin's Google register/login endpoints accepted JWTs without validating their signature, so an attacker could register or log in as any non-admin user whose email they knew.
- Technique / pattern: Crafted an unsigned JWT containing a target email and POSTed it to /wp-json/newspack-extended-access/v1/google/register; the server trusted the claims and authenticated the request.
- Takeaway: Third-party identity tokens must be signature-verified server-side against the provider's keys; reject unsigned/alg:none tokens and check audience/issuer.

### 2026-09-21 — 2FA Bypass leads to impersonation of legitimate users (Drugs.com) — n/a
- Source: [HackerOne #2885636](https://hackerone.com/reports/2885636)
- Type: 2FA bypass - session/trusted device survives email change
- Summary: An attacker could change their account email to a victim's address without re-triggering 2FA, keeping a trusted, authenticated session tied to the victim's identity.
- Technique / pattern: Registered with an attacker email, completed OTP and ticked 'trust this device', then changed the email to the victim's; no re-verification occurred, and access could be extended by cycling the email back and forth.
- Takeaway: Changing identity fields (email/phone) must require verification of the new value and invalidate trusted-device state and existing sessions.

### 2026-09-21 — 2FA bypass possible on authsvc.singlestore.com (SingleStore) — n/a
- Source: [HackerOne #3329361](https://hackerone.com/reports/3329361)
- Type: 2FA bypass - OTP brute force with ineffective lockout
- Summary: After three wrong MFA codes the service redirected (302) as if locked out, but a subsequent request with the correct code still authenticated, so the lockout gave no real protection against brute-forcing the numeric mfaToken.
- Technique / pattern: With valid credentials, intercepted the MFA submission and brute-forced the code in Burp Intruder, noticing that the 'locked' response was identical for correct and incorrect codes while the correct one still issued a session.
- Takeaway: A lockout must reject verification server-side, not just change the response. When testing, compare session/cookie issuance, not status codes alone.

### 2026-09-21 — No Rate Limiting on Password Attempts After Insecure Registration Flow cause ATO (Mars) — n/a
- Source: [HackerOne #3174778](https://hackerone.com/reports/3174778)
- Type: Missing rate limit on login -> brute-force ATO
- Summary: The login endpoint had no rate limiting, lockout or CAPTCHA, allowing unlimited password guessing against accounts identified through the registration flow (CWE-307).
- Technique / pattern: Identified valid accounts via registration behavior, then automated 100+ consecutive login attempts, using the appearance of a session cookie as the success signal.
- Takeaway: Rate limiting, progressive lockout and CAPTCHA on authentication endpoints are baseline controls; user enumeration plus no rate limit equals ATO.

### 2026-09-20 — Account takeover via an authentication bypass in the account-recovery flow (TikTok) — $12,000
- Source: [HackerOne #2443228](https://hackerone.com/reports/2443228)
- Type: Broken authentication (improper authentication mechanism in account recovery, Android)
- Summary: An improper authentication mechanism in TikTok's account-recovery process on Android could allow an attacker to take over another user's account. TikTok reported no evidence of exploitation in the wild and has fixed the issue.
- Technique / pattern: Recovery is the branch of the auth tree written to work when the user has lost their credentials, which is exactly the branch that must not trust anything the client supplies. Enumerate recovery per client — the mobile app, the web flow and any legacy endpoint frequently implement different step orders — then drive each step directly rather than through the UI, checking whether the server verifies the possession proof before issuing a session or only records that the client claims to have passed it.
- Takeaway: Map recovery separately for every client. A factor enforced in the web flow says nothing about the mobile flow that shares the same account store.

### 2026-09-20 — Web cache deception on abritel.fr turns a reflected session token into account takeover (Expedia Group) — n/a
- Source: [HackerOne #1698316](https://hackerone.com/reports/1698316)
- Type: Broken session management (session token reflected into a cacheable URL path)
- Summary: The session token was reflected in the URL path of a search route (`/search/keywords:.../minNightlyPrice/{anything}`) and the server answered with HTTP 200, which caused the caching layer to retain the response far longer than usual. A victim who loaded such a URL had a page containing their own session token stored in the shared cache, from where an attacker could retrieve it.
- Technique / pattern: Cache deception is the mirror of cache poisoning: rather than getting a payload into the cache, you get the *victim's authenticated response* into it. Append a path segment or a static-looking suffix that the application ignores but the cache treats as a cacheable asset, load it as an authenticated user, then request the same URL anonymously and diff. The status code matters — a 200 on a nonsense path is the signal that the app is routing loosely while the cache is keying strictly.
- Takeaway: Never reflect session material into a URL, and make the cache key and the authorization decision agree. Responses to authenticated requests should be explicitly marked private regardless of the path's shape.

### 2026-09-20 — Flickr account takeover by changing the Cognito email attribute (Flickr) — n/a
- Source: [HackerOne #1342088](https://hackerone.com/reports/1342088)
- Type: Broken authentication (identity keyed on the `email` claim instead of `sub`; `email_verified` ignored)
- Summary: Flickr's AWS Cognito-backed login identified accounts by the OIDC `email` claim rather than the immutable `sub`, ignored the `email_verified` claim, and normalised email case only on the client. An attacker could call Cognito's `update-user-attributes` API with their own access token to set their email to a case-variant of a victim's address, then sign in with that address and their own password to land in the victim's account.
- Technique / pattern: Where an app federates to an IdP, ask which claim it keys the local account on, whether that claim is mutable by the end user through the IdP's own API, and whether verification status is checked. Then bypass the web UI entirely and drive the IdP directly — the AWS CLI against Cognito, or the provider's user-attribute API — because the front end's validation (lowercasing, format checks, re-verification prompts) usually lives only in the browser. Case variants, Unicode look-alikes and plus-addressing are the standard probes for a normalisation gap.
- Takeaway: Key federated identities on the immutable subject identifier, require `email_verified`, and normalise server-side. Any claim the user can edit at the IdP is untrusted input to the relying party.

### 2026-09-20 — Account takeover via a session token issued before SMS verification (Zenly) — n/a
- Source: [HackerOne #1245762](https://hackerone.com/reports/1245762)
- Type: Broken authentication (deterministic pre-verification session token)
- Summary: The session-creation endpoint consistently returned the same not-yet-valid session token for a given user. An attacker could obtain that token in advance for a target phone number; once the legitimate owner later completed SMS verification, the previously issued token became valid and gave the attacker access to the account.
- Technique / pattern: Split every login into "token minted" and "token activated" and test them as separate states. Call the session-creation step twice for the same identity and compare the tokens — identical values mean the token is derived from the identity rather than from the attempt, so it can be fetched before the victim ever signs up. Then hold the token, complete verification from the victim side in a controlled test, and replay. Pre-registration variants of this (claiming an identity before its owner arrives) are the same bug seen from the other end.
- Takeaway: A session token must be unique per attempt and unusable until the factor completes — verification should mint a new token, never activate a previously handed-out one.

### 2026-09-19 — 0-click account takeover via a timed single-packet attack on the forgot-password flow (Mars) — n/a
- Source: [HackerOne #2142109](https://hackerone.com/reports/2142109)
- Type: Broken authentication (race condition in password-reset token issuance)
- Summary: The forgot-password feature could be attacked with carefully timed concurrent requests — a single-packet attack — to obtain a valid password-reset token for any account given only the victim's email address, yielding account takeover with no victim interaction.
- Technique / pattern: Where a flow generates a secret and mails it, look for state shared between concurrent requests: a token column overwritten per request, a cache key derived from something other than the user, a 'last generated token' read back by a second endpoint. HTTP/2's single-packet attack removes network jitter so requests land in the same millisecond, which turns theoretical races into reliable ones — Turbo Intruder and Burp's 'send group in parallel' are the standard harnesses. Test against accounts you control on both sides before claiming impact.
- Takeaway: Concurrency is an input. Any endpoint that mints, consumes or invalidates a secret needs testing in parallel, not just sequentially.

### 2026-09-19 — 2FA requirement not enforced when claiming bounties (HackerOne) — n/a
- Source: [HackerOne #2528919](https://hackerone.com/reports/2528919)
- Type: Broken authentication (policy enforced per workflow rather than per account)
- Summary: HackerOne programs can require two-factor authentication, and the platform enforced it in contexts such as embedded submissions, but the check was absent from the bounty-claim workflow — so a participant without 2FA could still complete that action.
- Technique / pattern: When a platform advertises a security *requirement* rather than a feature, treat the requirement as the specification and enumerate every workflow it should cover: submit, comment, claim payment, change payout details, accept an invitation, export data, use the API. Enforcement is normally attached to a handful of entry points by hand, so the list of covered flows and the list of sensitive flows drift apart over time.
- Takeaway: Enforce a policy at the account or session level so every workflow inherits it. A requirement implemented as per-endpoint checks will always have an endpoint that was missed.

### 2026-09-19 — Password reset token returned in the HTTP response of the reset request (Uber) — n/a
- Source: [HackerOne #173551](https://hackerone.com/reports/173551)
- Type: Broken authentication (secret disclosed in-band)
- Summary: Given only a valid account's email address, an attacker could trigger a password reset and read the reset token directly from the server's HTTP response, bypassing the email channel entirely and taking over the account.
- Technique / pattern: Whenever a flow's security depends on a secret reaching the user out of band, read the full response to the request that generates it — body, JSON fields, custom headers, redirect Location, and any subsequent polling or status call. Also check debug-shaped parameters and alternate formats (`.json`, mobile API versions), which frequently serialize the whole model including the token column that the HTML view omits.
- Takeaway: A token in the response defeats the channel it was supposed to travel through. Never serialize reset, verification or OTP values into any response.

### 2026-09-19 — 2FA OTP brute-forceable due to missing restrictions on failed attempts (Cloudflare Public Bug Bounty) — n/a
- Source: [HackerOne #1664974](https://hackerone.com/reports/1664974)
- Type: Broken authentication (no rate limit / lockout on OTP verification)
- Summary: The one-time-password step accepted unlimited failed attempts, so an attacker could brute-force the numeric code and eventually submit a correct one. Cloudflare's engineering team fixed it by restricting 2FA attempts.
- Technique / pattern: A six-digit code has a million values, but the practical question is the ratio of attempts allowed to code lifetime. Measure it: fire a few hundred wrong codes and watch for a `429`, a lockout, a CAPTCHA or silent invalidation of the code. Then re-test the weak spots — a new code issued mid-attack that does not reset the counter, a parallel mobile or API endpoint with no throttle, and counters keyed on IP rather than on account.
- Takeaway: The second factor is only as strong as the attempt limit behind it. Bind the counter to the account, invalidate the code after a small number of failures, and apply it on every endpoint that verifies it.

### 2026-09-19 — Authentication bypass via XML signature wrapping in SAML SSO (Rocket.Chat) — n/a
- Source: [HackerOne #3827674](https://hackerone.com/reports/3827674)
- Type: Broken authentication (SAML XSW, CVE-2026-58066)
- Summary: Rocket.Chat's SAML SSO verified an XML signature but did not bind the validated signature to the specific response and assertion the application then consumed. An unauthenticated attacker could wrap a forged assertion carrying arbitrary identity attributes alongside a legitimately signed element, and authenticate as any user (CVSS 9.8).
- Technique / pattern: The canonical SAML signature-wrapping attack. Capture one validly signed SAML response, keep the signed element byte-for-byte intact, and insert a second attacker-authored assertion elsewhere in the document (a sibling, a wrapper element, or inside an `Extensions`/`Object` node). The verifier locates and checks the signed node while the app's assertion-consumer reads the *other* one by id or by position — check-one-node, read-another. Vary where the injected assertion sits and how the two are referenced; different SP implementations resolve "which assertion" differently.
- Takeaway: Signature validation must operate on the exact element the application consumes, resolved by the same reference. Delegate SAML to a hardened library, pin the trusted assertion, and reject documents containing more than one assertion.

### 2026-09-19 — POST /api/bitcoinWithdrawalFees returns financial data without authentication (CoinMate.io) — n/a
- Source: [HackerOne #3676308](https://hackerone.com/reports/3676308)
- Type: Broken authentication (private endpoint missing its auth middleware)
- Summary: CoinMate documented `/api/bitcoinWithdrawalFees` as a private "USER OPERATION" requiring an HMAC-SHA256 signature (clientId, nonce, publicKey, signature), and all three official client libraries called it through their `postPrivate()` path — yet the server accepted an unauthenticated POST and returned live fee data, while every other private endpoint correctly rejected such requests. A permissive CORS policy (`Access-Control-Allow-Origin: *`) let any site read it cross-origin.
- Technique / pattern: Use the target's *own* documentation and SDKs as the specification for what should be authenticated, then test each private endpoint with all credentials stripped. Diffing "declared private" against "actually enforced" surfaces the single route where the auth middleware was never attached — a per-endpoint gap that a schema-wide assumption hides. The official client's `postPrivate()`/`postPublic()` split is a ready-made list of exactly which endpoints to re-test anonymously.
- Takeaway: Enforce authentication centrally so a route cannot be exposed by forgetting to annotate it, and reconcile the enforced set against the documented/SDK-private set. A wildcard CORS policy turns any such gap into silent cross-origin theft.

### 2026-09-19 — iOS Brave Playlist "Open in Private Tab" bypasses the FaceID gate on private tabs (Brave Software) — n/a
- Source: [HackerOne #3693295](https://hackerone.com/reports/3693295)
- Type: Broken authentication (local biometric gate not enforced on an alternate entry point)
- Summary: With Face ID protection enabled for private tabs, adding media to the Brave playlist and choosing "Open in a New Private Tab" from the playlist's context menu opened the private tab with no biometric or passcode prompt, exposing protected private-browsing content to anyone with physical device access.
- Technique / pattern: A biometric/passcode gate is usually wired to the *primary* way into a protected area and forgotten on every secondary route. Enumerate all the ways a protected surface can be reached — context menus, share sheets, deep links, widgets, Siri shortcuts, "open in", restored tabs after relaunch, notification taps — and exercise each one directly. The playlist "open in private tab" is one such side door; the gate lived on the private-tab switcher, not on the tab-creation primitive that the side door called.
- Takeaway: Attach the authentication check to the protected capability itself (creating/showing a private tab), so every caller inherits it, rather than to one UI path into it.

### 2026-09-19 — Session ID reuse allows XML-RPC API authentication bypass (Revive Adserver) — n/a
- Source: [HackerOne #3672641](https://hackerone.com/reports/3672641)
- Type: Broken authentication (session context not bound to the interface that issued it, CVE-2026-34917)
- Summary: Session identifiers minted for the web admin console were accepted by the XML-RPC API, which is meant to be administrator-only. A lower-privileged user could take their web-console session token and use it to reach the API, gaining access it should not have had (CVSS 4.3). The fix recorded whether a session originated from the web or the API and refused cross-context reuse.
- Technique / pattern: Where an application has more than one front door — web UI, XML-RPC/JSON-RPC API, mobile endpoint, GraphQL — check whether a session token issued by one is honored by the others, and whether the privilege ceiling differs between them. Log in through the lowest-privilege interface, then replay that exact session cookie/token against the higher-privilege API. Interfaces frequently share one session store while enforcing different authorization assumptions, so a token that means "logged-in user" on the web is silently read as "authorized API caller".
- Takeaway: Bind a session to the context that issued it and re-check privilege per interface. Sharing a session store across front doors requires each door to enforce its own authorization, not to trust that a valid session implies its own access level.

### 2026-09-18 — 2FA secret key could be changed without satisfying the 2FA verification requirement (HackerOne) — n/a
- Source: [HackerOne #1139535](https://hackerone.com/reports/1139535)
- Type: Broken authentication (second-factor re-enrollment without re-verification)
- Summary: Although disabling or editing two-factor authentication was supposed to require an OTP or a backup code, the 2FA secret key could be changed without providing that verification, letting anyone holding a session reconfigure the account's second factor.
- Technique / pattern: Enumerate every write operation on the second factor, not just 'disable': enroll, re-enroll, rotate the secret, regenerate backup codes, change the phone number, add a second device. Protection is commonly wired only to the disable button, so the rotate path reaches the same outcome by a different route.
- Takeaway: Any operation that replaces the second factor is equivalent to removing it, and must demand the current factor. Guard the capability, not one button.

### 2026-09-18 — Email OTP / 2FA bypass: session cookies issued at the OTP page before the code is validated (Drugs.com) — n/a
- Source: [HackerOne #2315420](https://hackerone.com/reports/2315420)
- Type: Broken authentication (session established before the second factor)
- Summary: The application issued its authentication cookies (such as PHPSESSID and bb_sessionhash) at the 2FA verification page itself rather than after the emailed OTP was validated, so the second factor could be skipped and the application accessed without it.
- Technique / pattern: After submitting valid first-factor credentials but before entering any code, inspect the cookie jar and replay an authenticated request directly to a post-login endpoint. If the session already carries full privilege, the OTP page is a UI step rather than a state transition.
- Takeaway: Do not mint an authenticated session until every required factor has passed. A pre-2FA session must be a distinct, unprivileged state that only the verified code can upgrade.

### 2026-09-18 — Two-factor authentication requirement bypassed, disclosing private program information (HackerOne) — n/a
- Source: [HackerOne #2486086](https://hackerone.com/reports/2486086)
- Type: Broken authentication (2FA requirement not enforced for a gated resource)
- Summary: The two-factor requirement guarding access to private program invitations and their content could be circumvented, so program information intended only for 2FA-enabled participants became reachable without genuinely satisfying that requirement.
- Technique / pattern: Where 2FA is used as an access condition for a resource rather than only for login, test the resource route independently of the enrollment flow: check whether the gate is re-evaluated on each request, whether a partially completed enrollment already flips the account's 'has 2FA' flag, and whether the underlying data endpoint applies the same condition as the page.
- Takeaway: A policy that says 'only 2FA-enabled accounts may see this' must be enforced at the data layer on every request. Enrollment state is not the same as an enforced, verified factor.

### 2026-09-18 — TOTP two-factor authentication bypassed by telling the server to use CAS during login (Rocket.Chat) — n/a
- Source: [HackerOne #1448268](https://hackerone.com/reports/1448268)
- Type: Broken authentication (alternate login handler skips the second factor)
- Summary: Rocket.Chat's 2FA login handler failed to validate the second factor when the login request asked the server to use CAS (Central Authentication Service). Including the CAS option in the request caused TOTP validation to be skipped and access to be granted.
- Technique / pattern: Enumerate every authentication method the server accepts - local password, LDAP, CAS, SAML, OAuth, token and API login - and drive each one directly rather than through the UI's default path. Second-factor enforcement is typically implemented inside one handler, so an alternate method reaching the same session issuance is a bypass by construction.
- Takeaway: Enforce the second factor at the point the session is issued, shared by all login handlers. Every additional auth method is another place the check must exist.

### 2026-09-18 — 2FA requirement bypass when inviting team members (Omise) — n/a
- Source: [HackerOne #3356149](https://hackerone.com/reports/3356149)
- Type: Broken authentication (client-side enforcement of a security policy)
- Summary: The application required 2FA to be enabled before a user could invite team members, but enforced that rule only in the frontend; flipping the boolean in the server's response let the invitation be sent with 2FA still disabled.
- Technique / pattern: Where the UI blocks an action, look at the response that drives the block. Intercepting proxies can rewrite responses automatically (Burp's match-and-replace) so the flag arrives as the client expects, and the subsequent request then reveals whether the server re-checks the policy or trusts the client's state.
- Takeaway: Any policy the client can observe, the client can lie about. Security requirements must be re-evaluated server-side at the moment of the privileged action.

### 2026-09-18 — 2FA bypass via response manipulation on the login page (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2962527](https://hackerone.com/reports/2962527)
- Type: Broken authentication (2FA bypass through response tampering)
- Summary: After valid credentials were submitted, the 2FA verification result was conveyed to the client as a numeric status in the response; changing it from the failure value to the success value completed login without the emailed one-time code.
- Technique / pattern: Submit a deliberately wrong OTP and diff the failure response against a known-good one. A small status field, boolean or code that differs is a client-side decision point; rewriting it shows whether the session is actually elevated server-side or whether the server merely advises the client of the outcome.
- Takeaway: Second-factor verification must change server-side session state. If the answer travels to the client as a status code, the second factor is advisory.

### 2026-09-18 — Session replay attack allows authentication bypass via captured login responses (WakaTime) — n/a
- Source: [HackerOne #3120790](https://hackerone.com/reports/3120790)
- Type: Broken session management (replayable session tokens, no invalidation)
- Summary: A captured successful-login response could be replayed in place of a failed one: submitting wrong credentials and substituting the earlier legitimate response granted account access, because session tokens stayed valid indefinitely and were not bound to the authentication event.
- Technique / pattern: Capture a full successful login, log out, then attempt a failed login and swap in the captured response. If access is granted, the tokens were never invalidated at logout and are not tied to the request that issued them - the same test also reveals whether lockout and rate-limit responses can be replaced away.
- Takeaway: Logout must invalidate server-side, and tokens should be short-lived and bound to their issuing context. A token that stays valid forever is a permanent credential harvested from any single capture.

### 2026-09-18 — SAML signature verification bypass allows unauthorized access (CVE-2024-6800) (GitHub) — n/a
- Source: [HackerOne #2579939](https://hackerone.com/reports/2579939)
- Type: Broken authentication (XML signature wrapping in SAML SSO)
- Summary: GitHub Enterprise Server, when configured with identity providers publishing signed federation metadata, was vulnerable to XML signature wrapping: an attacker with network access could forge a SAML response and authenticate as an arbitrary user, up to site administrator.
- Technique / pattern: In SAML flows, test whether the service provider verifies the signature over the element it actually reads. Signature wrapping works by keeping the validly signed fragment intact while adding a second, attacker-authored assertion elsewhere in the document - the verifier checks one node and the application consumes the other.
- Takeaway: SAML security depends on the verifier and the consumer resolving to the same element. Delegate signature validation to a hardened library and pin which assertion is trusted.

### 2026-09-18 — Two-factor authentication bypassed via account deactivation and password reset (HackerOne) — n/a
- Source: [HackerOne #2543342](https://hackerone.com/reports/2543342)
- Type: Broken authentication (2FA enforcement gap across an account state transition)
- Summary: A logical flaw in account recovery meant that after a user with 2FA enabled deactivated their account and then reactivated it through a password reset, the subsequent login did not enforce the second factor.
- Technique / pattern: Map every state an account can be in — active, deactivated, locked, recovering, newly reactivated — and re-test the second factor from each; enforcement is usually implemented on the normal login path only, and a state transition is what skips it.
- Takeaway: 2FA is a property of the account, not of one code path: any flow that can produce a valid session — recovery, reactivation, SSO linking, impersonation — has to re-assert the second factor.

### 2026-09-18 — Password not verified when disabling 2FA with a backup code (HackerOne) — n/a
- Source: [HackerOne #587910](https://hackerone.com/reports/587910)
- Type: Broken authentication (missing re-authentication on a security-settings change)
- Summary: The disable-2FA request asked for both a password and a backup code, but the server validated only the backup code: submitting an arbitrary password alongside a valid backup code successfully turned 2FA off.
- Technique / pattern: When a sensitive form collects more than one secret, test each field independently — send a deliberately wrong value in one while keeping the other valid; a success proves the field is decorative and never reaches a check.
- Takeaway: A field on the form is not a check on the server. Security-settings changes need genuine re-authentication, and every credential the UI collects should be verified or not asked for at all.

### 2026-09-18 — Two-factor authentication enforcement bypassed through group assignment (Nextcloud) — n/a
- Source: [HackerOne #1050244](https://hackerone.com/reports/1050244)
- Type: Broken authentication (group-scoped 2FA policy bypass)
- Summary: Nextcloud's mandatory-2FA policy is applied per user group; the reported flow showed that account/group manipulation let an account end up outside the enforcing group, so it could log in without satisfying the enforced second factor.
- Technique / pattern: Where a security policy is scoped to a group, role or tag, attack the membership rather than the policy: check who can create groups, who can move a user between them, and whether enforcement is re-evaluated at login or only when the policy is set.
- Takeaway: A policy applied to a container is only as strong as the rules on joining and leaving that container — enforcement must be evaluated against the user's state at authentication time, not assumed from configuration.

### 2026-09-18 — OAuth redirect_uri bypass using an IDN homograph domain leaks access tokens (Semrush) — n/a
- Source: [HackerOne #861940](https://hackerone.com/reports/861940)
- Type: Broken authentication (OAuth redirect_uri validation bypass)
- Summary: The OAuth implementation failed to validate `redirect_uri` strictly; a homograph domain built from visually similar Unicode characters passed the check, so the authorization flow delivered the user's access token to an attacker-controlled domain.
- Technique / pattern: Probe redirect validation with equivalence tricks rather than obviously foreign URLs: IDN/punycode homographs, added subdomains and paths, trailing characters, alternate encodings — anything that a normalising comparison treats as equal to the registered value.
- Takeaway: `redirect_uri` must be compared against an exact registered value after normalisation (and punycode-decoded before display), because a validator that reasons about how a domain *looks* will eventually be shown one that looks right and is not.

### 2026-09-18 — Authentication bypass leading to zero-interaction account takeover (Krisp) — n/a
- Source: [HackerOne #1608151](https://hackerone.com/reports/1608151)
- Type: Broken authentication (auth bypass, 0-click account takeover)
- Summary: An authentication bypass on a Krisp endpoint allowed full account takeover with no interaction required from the victim.
- Technique / pattern: Map every authentication entry point, not just the login form, and test whether any accepts identity or state the client controls — a no-interaction bypass almost always means the server trusts a request-supplied identifier instead of re-deriving it from verified credentials.
- Takeaway: Any auth flaw needing zero victim interaction is critical by construction; the entire auth surface must derive identity server-side from proven credentials.

### 2026-09-18 — Authentication bypass of the email verification code at signup (UPchieve) — n/a
- Source: [HackerOne #1406471](https://hackerone.com/reports/1406471)
- Type: Broken authentication (email verification / OTP gate bypass)
- Summary: The registration flow's email verification code could be bypassed, letting an attacker complete signup and reach a verified state without controlling the email address.
- Technique / pattern: Test verification gates for response manipulation, missing server-side state, reusable or guessable codes, and — most often — simply calling the post-verification endpoint directly and skipping the step.
- Takeaway: Email/OTP verification must be enforced server-side as a state transition; if the 'verified' flag is reachable without presenting the code, the control is decorative.

### 2026-09-18 — Bypass password authentication to update the account password (X / xAI) — n/a
- Source: [HackerOne #970157](https://hackerone.com/reports/970157)
- Type: Broken authentication (step-up re-authentication bypass)
- Summary: Twitter's requirement to re-enter the current password before changing it — a control specifically meant to limit the damage of a hijacked session — could be bypassed, so a session holder could set a new password without knowing the old one.
- Technique / pattern: For every 'confirm your password' step, verify the server actually validates it: drop the parameter, send it empty, replay the request without it, or call the underlying update endpoint directly.
- Takeaway: Step-up re-authentication is only as strong as its server-side check; a bypass converts a temporarily stolen session into a permanent account takeover.

### 2026-09-18 — Authorization bypass using login by OTP code on the mobile API (Grab) — n/a
- Source: [HackerOne #205000](https://hackerone.com/reports/205000)
- Type: Broken authentication (mobile login endpoint requiring fewer factors than the web)
- Summary: The Grab Android app's login endpoint accepted authentication with only the SMS OTP and no password, so the code alone was sufficient to take over an account (the victim did receive an SMS notification).
- Technique / pattern: Compare the mobile API's auth flow against the web flow factor by factor — mobile endpoints frequently implement a reduced set — then determine which factors the server actually enforces rather than which the UI collects.
- Takeaway: Authentication requirements must be identical across every client; test each mobile or alternate API login endpoint independently instead of assuming it inherits the web application's checks.

### 2026-09-17 — Account Takeover via Password Reset without user interactions (CVE-2023-7028) (GitLab) — $35,000
- Source: [HackerOne #2293343](https://hackerone.com/reports/2293343)
- Type: Broken authentication (password-reset delivery to attacker-controlled address)
- Summary: GitLab's password-reset flow could deliver the reset link to an unverified, user-supplied secondary address, so submitting a victim's account together with an attacker address yielded a valid reset token and full takeover with no victim interaction.
- Technique / pattern: Test whether the reset endpoint accepts more than one destination address, and whether the delivery target is re-derived from the stored, verified account email rather than from request input.
- Takeaway: Password reset is the real authentication boundary: the delivery address must come from verified server-side state, never from anything in the request.

### 2026-09-17 — Bypassing HackerOne 2FA due to race condition (HackerOne) — n/a
- Source: [HackerOne #2598548](https://hackerone.com/reports/2598548)
- Type: Broken authentication (2FA bypass via race condition)
- Summary: Concurrent requests against the two-factor verification step allowed the second factor to be bypassed because validation and session upgrade were not performed atomically.
- Technique / pattern: Capture the OTP submission and fire many copies in parallel (single-packet / synchronized request techniques) to land requests in the window between the check and the state update.
- Takeaway: Authentication state transitions need atomicity and strictly single-use tokens — test 2FA, coupon and limit checks for concurrency, not only for logic flaws.

### 2026-09-17 — Account takeover of existing HackerOne accounts through SCIM provisioning (HackerOne) — n/a
- Source: [HackerOne #3178999](https://hackerone.com/reports/3178999)
- Type: Broken authentication (SSO / SCIM identity binding)
- Summary: SCIM provisioning could bind an existing platform account to an identity-provider identity controlled by someone else, handing over control of a pre-existing account.
- Technique / pattern: Examine how the provisioning layer matches an incoming IdP identity to a local user — usually by email — and whether that match requires prior verification or proof of domain ownership before it takes effect.
- Takeaway: Enterprise provisioning (SCIM / SAML / JIT) is a full authentication path with its own trust assumptions; an org admin must never be able to claim identities they do not own.

### 2026-09-17 — Account takeover via improper JWT signature validation (Linktree) — n/a
- Source: [HackerOne #1760403](https://hackerone.com/reports/1760403)
- Type: Broken authentication (JWT verification flaw)
- Summary: The application accepted JSON Web Tokens without properly verifying their signature, so an attacker could alter or forge the token's claims and authenticate as another user.
- Technique / pattern: Decode the token and test the classic verification failures in order: alg:none, algorithm confusion (RS256 verified as HS256 using the public key as the HMAC secret), unvalidated kid handling, and outright signature stripping.
- Takeaway: Pin the expected algorithm and key server-side — a JWT library that 'verifies' using attacker-chosen header parameters is not verifying anything at all.

### 2026-09-17 — 1-Click Account Takeover via whitelisted subdomain redirect (Hostinger) — n/a
- Source: [HackerOne #3081691](https://hackerone.com/reports/3081691)
- Type: Broken Authentication / Session (open redirect -> token theft)
- Summary: marketing.hostinger.com was whitelisted for redirects; an open redirect there let an attacker craft a single link that bounced the authenticated redirect (carrying a token) to an attacker destination, yielding full account takeover.
- Technique / pattern: Abused the redirect whitelist on a trusted subdomain to send an auth redirect to an attacker-controlled URL, capturing the token with one victim click.
- Takeaway: A whitelisted/trusted subdomain with an open redirect is as dangerous as one on the main domain — validate redirect targets strictly and never route tokens through redirectable URLs.

### 2026-09-17 — Account takeover due to email-change flow (Mattermost) — n/a
- Source: [HackerOne #1114347](https://hackerone.com/reports/1114347)
- Type: Broken Authentication (email change / account recovery)
- Summary: A flaw in the email-change process left an account takeoverable when a pending, unverified email change interacted badly with the account-recovery mechanism.
- Technique / pattern: Walked the email-change lifecycle — request a change to a new address, then abused the recovery/verification gap around the pending (unconfirmed) email to gain access.
- Takeaway: Treat email change as a security-critical flow — require verification of the new address, invalidate the change if unconfirmed, and don't let a pending change weaken recovery.

### 2026-09-17 — OTP verification bypass via client-side response manipulation (bitaccess) — n/a
- Source: [hackerone #130460](https://hackerone.com/reports/130460)
- Type: Broken Authentication (OTP / 2FA bypass)
- Summary: OTP validation depended on a server response the client could tamper with, so flipping the "invalid" result to "valid" bypassed the one-time-password step.
- Technique / pattern: Intercept the OTP verification response and change the failure indicator to success where the client trusts the response to decide whether the OTP passed.
- Takeaway: Authentication decisions must be enforced and re-checked server-side; never let the client's interpretation of a response determine whether an OTP/2FA step succeeded.

### 2026-09-17 — OTP bypass — one-time code exposed in the API response (MTN Group) — n/a
- Source: [hackerone #777957](https://hackerone.com/reports/777957)
- Type: Broken Authentication (OTP exposure / in-band secret)
- Summary: The subscription/account flow returned the OTP value inside the HTTP response, so an attacker inspecting network traffic could read the code and complete authentication to modify another subscriber's account.
- Technique / pattern: Trigger the OTP for a target number, read the OTP directly from the API response body, then submit it to authorize the protected action.
- Takeaway: Never return OTPs or other verification secrets in responses — deliver them only out-of-band (SMS/email) and validate them solely server-side.

### 2026-09-16 — Session cookie not invalidated after logout (Coursera) — n/a
- Source: [hackerone #152080](https://hackerone.com/reports/152080)
- Type: Broken Authentication & Session Management
- Summary: Coursera failed to invalidate session cookies on logout, so a previously captured cookie stayed valid (up to ~24 hours) and could be replayed to access and modify the account.
- Technique / pattern: Capture a session cookie, log out, then replay the old cookie against authenticated endpoints; if it still works, server-side session invalidation is missing.
- Takeaway: Logout must destroy the session server-side, not just clear the client cookie; tie sessions to server state with proper expiry and rotation.

### 2026-09-16 — Password-reset / session-token flaw enabling account takeover (Courier) — n/a
- Source: [hackerone #948345](https://hackerone.com/reports/948345)
- Type: Broken Authentication & Session Management
- Summary: A weakness in Courier's password-reset / session-token handling let an attacker regain account access without properly consuming the emailed reset code, enabling account takeover.
- Technique / pattern: Analyze the reset flow and its tokens/session state for codes that aren't strictly bound to one account, single-use, or validated server-side before an authenticated session is issued.
- Takeaway: Reset tokens must be single-use, time-bound, and server-validated, and no authenticated session should be issued until the code is verified.

### 2026-09-15 — 2FA bypass by sending blank code (Glassdoor) — n/a
- Source: [HackerOne #897385](https://hackerone.com/reports/897385)
- Type: Broken authentication (2FA/OTP bypass)
- Summary: Glassdoor's two-factor verification could be bypassed by submitting an empty/blank code, due to a missing null check on the entered value during validation.
- Technique / pattern: At the OTP prompt, submit an empty code value; flawed comparison logic treated the blank input as valid (or skipped validation), completing authentication.
- Takeaway: OTP/2FA validation must explicitly reject empty, null, and malformed input and compare only against a freshly generated server-side code.

### 2026-09-15 — 2FA bypass — confirmation tokens do not expire (GSA Bounty) — n/a
- Source: [HackerOne #264090](https://hackerone.com/reports/264090)
- Type: Broken authentication (token lifecycle / 2FA bypass)
- Summary: Email confirmation tokens issued during sign-up did not expire, letting an attacker reuse a captured token to bypass the platform's account lockdown/verification controls.
- Technique / pattern: Capture a confirmation/registration token from the sign-on email and replay it after the intended validity window; the server never time-bounded or invalidated the token.
- Takeaway: Authentication and confirmation tokens must be single-use and time-limited, and invalidated after use or lockout — non-expiring tokens defeat rate-limiting and 2FA protections.

### 2026-09-15 — Bypass email verification in the OAuth flow (GitLab) — n/a
- Source: [HackerOne #922456](https://hackerone.com/reports/922456)
- Type: Broken Authentication (email verification bypass / OAuth)
- Summary: GitLab required a verified email before completing an OAuth flow, but the check was bypassable, so an unverified email could be treated as verified during authentication.
- Technique / pattern: Probe the ordering of email verification vs OAuth linking; a step that assumes prior verification but does not re-check lets an attacker proceed with an unverified (or attacker-set) email, undermining email-as-identity on downstream services.
- Takeaway: Re-verify security preconditions at the step that depends on them; never infer "email verified" from a flow the user can influence.

### 2026-09-15 — Misconfigured OAuth leads to pre-account takeover (Bumble / Badoo) — n/a
- Source: [HackerOne #1074047](https://hackerone.com/reports/1074047)
- Type: Broken Authentication (OAuth pre-account-takeover)
- Summary: Because signup via multiple identity providers did not check whether an account already existed for an email, an attacker could pre-create an account with the victim's email so that when the victim later signed in via OAuth their session bound to the attacker-controlled account.
- Technique / pattern: Register the victim's email through one path/provider before they sign up; when the victim authenticates via a social provider that trusts the email without linking-verification, the identities merge into the pre-existing (attacker) account.
- Takeaway: On OAuth/social login, verify email ownership and reconcile against existing accounts before auto-linking; pre-provisioned accounts are a stealthy ATO vector.

### 2026-09-14 — 2FA OTP not invalidated (reusable OTP) (HackerOne) — n/a
- Source: [HackerOne #2529780](https://hackerone.com/reports/2529780)
- Type: Broken authentication — improper 2FA/OTP handling
- Summary: TOTP codes for two-factor auth were not invalidated after use or after a newer code was generated, so a previously issued OTP stayed valid within its window and could be replayed.
- Technique / pattern: After enabling TOTP, capture a valid code and test whether it still authenticates after a new code is generated or after first use; success indicates missing single-use/replay protection.
- Takeaway: Treat OTPs as single-use — invalidate a code once consumed and expire prior codes; enforce server-side replay protection within the time step.

### 2026-09-14 — Two-factor authentication bypass on Android app endpoint (Grab) — n/a
- Source: [HackerOne #202425](https://hackerone.com/reports/202425)
- Type: Broken authentication — 2FA bypass
- Summary: An endpoint used by the Grab Android app allowed the two-factor step to be bypassed, exposing accounts to takeover; the researcher linked it to a related unauthenticated-ATO issue.
- Technique / pattern: Test mobile/alternate API endpoints separately from the web flow — a 2FA gate enforced on the website may be missing or skippable on the app's API surface.
- Takeaway: Enforce 2FA consistently across every client and endpoint (web, mobile, legacy APIs), not just the primary login UI.


### 2026-09-13 — Authentication bypass: backend over-trusts submitted login data (MTN Group) — n/a
- Source: [HackerOne #1709881](https://hackerone.com/reports/1709881)
- Type: Broken authentication (auth bypass / mass ATO)
- Summary: The login backend placed too much trust in client-submitted login information, letting a remote attacker authenticate as any user even with the wrong password — reportedly around 100,000 accounts.
- Technique / pattern: Inspect the login flow for client-controlled fields that decide auth success (status flags, user ids, "verified" booleans) and tamper them to see whether the server re-validates server-side.
- Takeaway: The server must derive authentication state solely from credentials it verifies itself — never from values the client can set.

### 2026-09-13 — Account takeover via disclosed session cookie (HackerOne) — n/a
- Source: [HackerOne #745324](https://hackerone.com/reports/745324)
- Type: Broken session management (session disclosure)
- Summary: A valid session cookie for a Security Analyst account was disclosed through human error, allowing an outside party to access the account directly.
- Technique / pattern: Session cookies are bearer tokens — whoever holds one is authenticated; they leak via screenshots, support tickets, logs, and shared debug output.
- Takeaway: Bind sessions to additional signals and expire them aggressively; treat any session-token exposure as full compromise and rotate/invalidate immediately.

### 2026-09-12 — No rate limit on 2FA code during password reset -> 2FA bypass (Slack) — n/a
- Source: [HackerOne #121696](https://hackerone.com/reports/121696)
- Type: Broken authentication (2FA / rate limiting)
- Summary: The password-reset flow allowed many incorrect two-factor codes to be submitted without effective lockout, opening the second factor to brute force.
- Technique / pattern: Repeatedly submitted wrong 2FA verification codes on the reset page and observed no rate limiting or lockout, making the finite code space guessable.
- Takeaway: Apply strict rate limiting and lockout to every code-verification step (including recovery flows), not just the primary login.

### 2026-09-12 — Sessions not invalidated on password change / logout (Omise) — n/a
- Source: [HackerOne #634488](https://hackerone.com/reports/634488)
- Type: Broken authentication / session management
- Summary: After a password change or logout in one browser, an existing session in another browser stayed alive, so a compromised session persisted despite the user's remediation.
- Technique / pattern: Logged in from two browsers, changed the password / logged out in one, and confirmed the other session remained authenticated.
- Takeaway: Invalidate all active sessions server-side on password change and logout; client-side logout alone does not revoke a stolen session.

### 2026-09-11 — No rate-limit on password-reset verification -> ATO (Clario) — n/a
- Source: [HackerOne #767765](https://hackerone.com/reports/767765)
- Type: Broken authentication — missing rate-limit on reset OTP -> brute force
- Summary: The password-reset verification endpoint (POST /v1/verification-code/forgot-password) had no rate limiting, so knowing only a victim's email an attacker could brute-force the reset verification code and take over the account.
- Technique / pattern: Locate the reset-code verification request, then automate high-volume guesses of the code for a target email; absent lockout/throttling, the finite code space is exhausted to complete the reset.
- Takeaway: Reset/OTP verification endpoints must enforce strict rate-limiting, lockout, and sufficient code entropy/expiry — an unthrottled code check is a direct path to ATO.

### 2026-09-11 — Reset-link path parameter manipulation leaks token -> ATO (Mars) — n/a
- Source: [HackerOne #2341038](https://hackerone.com/reports/2341038)
- Type: Broken authentication — password-reset link redirection / token leakage
- Summary: The password-reset email link contained a parameter specifying the reset page's path; an attacker could modify it so clicking the link sent the victim (and the reset token) to an attacker-controlled domain.
- Technique / pattern: Inspect the reset link for host/path parameters influenced by the request (host-header/path poisoning), tamper the parameter to point at an attacker origin so the token is delivered off-site when the victim clicks, then use it to reset the password.
- Takeaway: Build reset links from a server-side fixed base URL — never from client-controllable host/path parameters — so the token can't be redirected to an attacker.

### 2026-09-11 — Account Takeover in Password Reset, no OTP needed (Mars) — n/a
- Source: [HackerOne #3228888](https://hackerone.com/reports/3228888)
- Type: Broken authentication — password reset / OTP bypass → 0-interaction ATO
- Summary: A flaw in the "Forgot Password" flow let an attacker take over any account without access to the victim's phone number or the SMS OTP.
- Technique / pattern: Abuse the reset endpoint's trust in client-controlled data — the OTP was not properly validated server-side (response/step manipulation), so the reset could be completed without the genuine code.
- Takeaway: Validate reset OTPs/tokens server-side and bind them to the requesting session; never trust client-side success signals or skippable steps in the reset flow.

### 2026-09-11 — Authentication bypass via username enumeration + login brute force (Automattic) — n/a
- Source: [HackerOne #209008](https://hackerone.com/reports/209008)
- Type: Broken authentication — auth-bypass chain
- Summary: A WordPress JSON login endpoint (en.instagram-brand.com) allowed username enumeration and had no brute-force protection, chaining into account takeover.
- Technique / pattern: Enumerate valid usernames through differential login responses, then brute-force passwords against the unthrottled endpoint to bypass authentication.
- Takeaway: Return uniform responses to hide username validity and enforce rate-limiting/lockout on login — enumeration plus no rate limit equals practical ATO.

### 2026-09-10 — Full account takeover via shared password-reset token (UPchieve) — n/a
- Source: [HackerOne #1175081](https://hackerone.com/reports/1175081)
- Type: Broken authentication — password reset
- Summary: Tampering with the email parameter in the forgot-password request caused the same reset token to be delivered to multiple email addresses, so an attacker's inbox received a token valid for the victim's account.
- Technique / pattern: In the reset request, submit multiple/array emails (attacker + victim) via Burp; if a token landing in the attacker inbox validates for the victim, reset their password.
- Takeaway: Reset tokens must bind to exactly one account and one request; never accept multiple recipients or trust a client-supplied email list.

### 2026-09-10 — Reset-password cookie leads to account takeover (Weblate) — n/a
- Source: [HackerOne #1004536](https://hackerone.com/reports/1004536)
- Type: Broken authentication / session handling in reset flow
- Summary: The password-reset flow stored reset state in a cookie; reusing the reset link (for example after closing the browser) mishandled that state and enabled account takeover through the reset mechanism.
- Technique / pattern: Test the reset-link lifecycle — reuse after first click, after browser close — and inspect any reset cookie/state for tokens that aren't invalidated on use or that bind to the wrong session.
- Takeaway: Reset links must be strictly single-use and invalidated immediately; don't persist reset authorization in a reusable cookie.

### 2026-09-09 — 0-click account takeover (Remitly) — Critical
- Source: [HackerOne #2831902](https://hackerone.com/reports/2831902)
- Type: Account takeover (0-click) in the authentication flow
- Summary: A disclosed **zero-interaction account takeover** — the auth/identity flow let an attacker take over an account without any victim action. (See the report for the exact chain.)
- Technique / pattern: 0-click ATOs usually chain a predictable/leaked identifier or a flow that binds credentials/sessions to attacker-controlled input (reset link, OTP, OAuth `state`/email binding) without verifying the victim.
- Takeaway: Map the *entire* auth flow (signup, reset, OAuth, email change, 2FA) and look for any step that trusts attacker-supplied data to identify the account.

### 2026-09-09 — Insecure password reset → account takeover (Vimeo) — $5,000
- Source: [HackerOne #42587](https://hackerone.com/reports/42587)
- Type: Broken authentication — password reset
- Summary: A weakness in the **password-reset** flow allowed resetting another user's password (reset token not properly bound / one-time / expiring), leading to ATO.
- Technique / pattern: Inspect reset tokens for predictability, reuse, missing expiry, or lack of binding to the user/request; test host-header poisoning of the reset link and token-to-account confusion.
- Takeaway: Reset tokens must be high-entropy, single-use, expiring, and bound to the exact account + request. Password reset is the #1 ATO surface.
