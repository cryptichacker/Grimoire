---
tags: [hacking, bug-bounty, authentication, session, ato, reports]
type: log
---

# Broken Authentication

## Up
- [[Reports]]

Disclosed **broken authentication & session management** reports — account takeover via password-reset flaws, OAuth misuse, 2FA bypass, weak/identifiable tokens, session fixation. See [[Pattern]] → Broken Authentication.

## Reports

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
