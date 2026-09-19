---
tags: [hacking, web, portswigger, authentication]
type: vulnerability-category
source: https://portswigger.net/web-security/authentication
last-verified: 2026-08-25
---

# Authentication

## Up
- [[PortSwigger]]

## What it is
Authentication is the process of verifying that a user is who they claim to be. It differs from **authorization** (what an authenticated user is allowed to do). The three common factors are: something you *know* (password, security question), something you *have* (a phone or hardware token), and something you *are* (biometrics). Authentication vulnerabilities let an attacker either brute-force their way in or bypass the mechanism entirely ("broken authentication").

## How the vulnerabilities arise
Two broad causes:
1. **Weak brute-force protection** — the mechanism doesn't adequately stop an attacker guessing credentials.
2. **Logic flaws / broken implementation** — coding mistakes let an attacker skip or subvert checks.

## Impact
Compromising authentication gives access to the target account's data and functionality. Breaching a high-privilege account can mean full control of the application; even a low-privilege account may expose sensitive data and widen the attack surface (internal pages otherwise unreachable).

## Password-based login

### Brute-forcing usernames
Usernames are guessable when predictable: email patterns (`firstname.lastname@company.com`), high-privilege defaults (`admin`, `administrator`), or names disclosed in public profiles/HTTP responses.

### Brute-forcing passwords
Password policies don't stop attacks because users pick predictable variations — `mypassword` → `Mypassword1!`, and on forced rotation `Mypassword1?` → `Mypassword2!`. Effective attacks exploit human patterns, not random combinations.

### Username enumeration
Detect valid usernames from any observable difference between valid and invalid submissions:
- **Status codes** — a different HTTP code for valid usernames.
- **Error messages** — "invalid username" vs "incorrect password" (even a one-character or invisible difference leaks it).
- **Response timing** — a valid username triggers extra backend work (e.g. password hashing) and responds slower; amplify by sending an excessively long password.

### Flawed brute-force protection
- **Account lockout** can be bypassed: if the failed-attempt counter resets on any successful login, periodically insert your own valid login into the wordlist; or distribute attempts — with a 3-attempt limit, try 3 passwords against 1000 usernames instead of 1000 against one.
- **Credential stuffing** slips past lockout because each account is tried only once.
- **IP-based rate limits** can be bypassed by spoofing/rotating the source IP (e.g. `X-Forwarded-For`), by supplying multiple passwords in a single request where the logic allows it, or by simply waiting out a time-based block.

### HTTP Basic authentication
Sends `Authorization: Basic base64(username:password)` on every request. The token is static, so it's exposed to interception (MITM without HSTS), rarely has brute-force protection, offers no CSRF protection, and the credentials may be reused elsewhere.

## Multi-factor authentication

### Bypassing 2FA
If the app grants a logged-in session after the first factor and only *then* prompts for the code, the second step may not be enforced — try loading a logged-in-only page directly without submitting the code.

### Flawed 2FA logic
If the account being verified is carried in a cookie you control — e.g. `Set-Cookie: account=carlos` set at step one and read at step two — change it to another user's name and submit *your own* code to log in as them:
```
POST /login-steps/second HTTP/1.1
Cookie: account=victim-user
verification-code=123456
```

### Brute-forcing verification codes
4–6 digit codes have a small space. "Log the user out after N wrong codes" fails if the whole multi-step flow can be automated (Burp Intruder macros / Turbo Intruder) to keep re-establishing the session. SMS codes also face interception and SIM-swap.

## Other mechanisms

### "Remember me" / stay-logged-in cookies
Weak when built from predictable values (username + timestamp), merely Base64/encoded, or unsalted-hashed (crackable by hashing a wordlist, or via rainbow tables). Stealing one (e.g. via XSS) or reverse-engineering the formula (easy if the framework is open source) lets an attacker forge others.

### Password reset
- **User-in-URL** is insecure: `reset-password?user=victim-user` lets you swap the target. Secure resets use a high-entropy, short-lived token: `reset-password?token=<random>`.
- **Token not re-validated** — if the reset form doesn't check the token again on submit, delete it and reset an arbitrary account.
- **Password reset poisoning** — if the reset URL's domain is built from the `Host` (or `X-Forwarded-Host`) header, poison it so the victim's token is sent to an attacker server.

### Password change
Uses the same checks as login and shares its flaws. If the username is in a hidden field, editing it can target other users.

## Prevention
- Enforce strong, unique credentials and check against breached-password lists.
- Return identical, generic responses/timing regardless of whether the username exists.
- Robust, layered brute-force protection (rate limits + lockout + CAPTCHA) that also defends distributed/credential-stuffing patterns.
- Verify MFA properly: enforce every step server-side, tie the code to the authenticated session (not a client-controlled value), and rate-limit codes.
- Use secure, high-entropy, single-use, short-lived reset tokens; never build security-relevant URLs from the `Host` header.

## Labs
14 labs. Titles from the academy learning path — verify exact wording against the live page; "General approach" is the generic technique.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Username enumeration via different responses | Apprentice | Compare error messages between valid/invalid usernames, then brute-force the password. | |
| 2FA simple bypass | Apprentice | Log in, then skip straight to the logged-in page without submitting the 2FA code. | |
| Password reset broken logic | Apprentice | Reset form doesn't validate the token/username; change the username to reset the victim's password. | |
| Username enumeration via subtly different responses | Practitioner | Diff responses closely (a trailing space / punctuation) to find the valid username. | |
| Username enumeration via response timing | Practitioner | Long password + timing difference reveals valid usernames; bypass IP block with `X-Forwarded-For`. | |
| Broken brute-force protection, IP block | Practitioner | Reset the failed-attempt counter by interleaving a successful login of your own account. | |
| Username enumeration via account lock | Practitioner | Trigger lockout behaviour that differs for valid usernames, then brute-force. | |
| 2FA broken logic | Practitioner | Change the account cookie at the 2FA step to target another user; brute-force/deliver their code. | |
| Brute-forcing a stay-logged-in cookie | Practitioner | Reverse the cookie formula (e.g. base64 `user:md5(password)`) and brute-force the password. | |
| Offline password cracking | Practitioner | Steal the stay-logged-in cookie via stored XSS, crack the password hash offline. | |
| Password reset poisoning via middleware | Practitioner | Inject `X-Forwarded-Host` so the reset link points at your server, capturing the victim's token. | |
| Password brute-force via password change | Practitioner | Abuse the change-password form (different responses / no rate limit) to brute-force. | |
| Broken brute-force protection, multiple credentials per request | Expert | Send passwords as a JSON array in one request to bypass per-request rate limiting. | |
| 2FA bypass using a brute-force attack | Expert | Automate the full login flow (macro) to keep the session valid while brute-forcing the code. | |
