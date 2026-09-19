---
tags: [hacking, web, portswigger, race-conditions]
type: vulnerability-category
source: https://portswigger.net/web-security/race-conditions
last-verified: 2026-08-25
---

# Race conditions

## Up
- [[PortSwigger]]

## What it is
A race condition occurs when a website processes requests concurrently without adequate safeguards, so multiple threads act on the same data at the same time and collide, producing unintended behaviour. The brief exploitable interval is the **race window**. Most are a form of **TOCTOU** (time-of-check to time-of-use): the gap between a check and the action that relies on it.

## Limit overrun (the classic class)
The app checks a condition, acts, then updates state — but two requests slip through the check before the update lands. Examples:
- Redeeming a gift card / discount code more than once.
- Rating a product multiple times, withdrawing more than the balance, reusing a CAPTCHA solution, bypassing an anti-brute-force limit.

## Hidden multi-step sub-states
A single HTTP request can drive a multi-step back-end process that transitions through temporary sub-states. Colliding requests can catch the object mid-construction — e.g. establishing an authenticated session *before* `enforce_mfa` is set, bypassing MFA.

## Detection & exploitation techniques
- **Single-packet attack (HTTP/2)** — Burp sends 20–30 requests in one TCP packet so they arrive together, neutralising network jitter. This is the go-to technique.
- **Last-byte synchronization (HTTP/1)** — send all requests but withhold the final byte, then release them together.
- **Turbo Intruder** — gate requests and open the gate to fire them in parallel; use for retries/timing control.

## Methodology
1. **Predict collisions** — target security-critical endpoints where multiple requests touch the same record. (Different users resetting the *same* account is exploitable; separate accounts aren't.)
2. **Probe for clues** — benchmark sequential behaviour, then fire parallel requests and look for *any* deviation, including second-order effects like different email contents.
3. **Prove & refine** — reproduce reliably, strip unnecessary requests, understand the structural weakness.

## More categories
- **Multi-endpoint races** — race the window between validation and confirmation across two endpoints (e.g. add an item to the cart between payment validation and order confirmation). Use connection warming to align timing.
- **Single-endpoint races** — parallel requests with different values to one endpoint (e.g. two password resets with different usernames may bind the token to one user but email it to another). Background email threads are ideal.
- **Partial construction** — an object built across several statements has an uninitialized window; inject a value matching the empty/null state, e.g. `GET /api/user/info?user=victim&api-key[]=`.
- **Session-based locking** — frameworks that serialize per session (PHP) can be bypassed by using a **different session token** per request.
- **Time-sensitive** — tokens derived from a high-res timestamp instead of randomness collide if two requests are timed to the same instant.

## Prevention
- Perform state-changing operations in a **single atomic** database transaction.
- Use datastore concurrency features and **uniqueness constraints**.
- Don't split security state across storage layers (e.g. session guarding a DB operation).
- Keep the session framework internally consistent; consider stateless encrypted state (JWT).

## Labs
6 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Limit overrun race conditions | Apprentice | Single-packet attack to redeem a discount/coupon multiple times. | |
| Bypassing rate limits via race conditions | Practitioner | Fire login attempts in one packet to beat the rate limiter. | |
| Multi-endpoint race conditions | Practitioner | Race add-to-cart against checkout to buy above your balance. | |
| Single-endpoint race conditions | Practitioner | Parallel email-change/confirm to bind an address you don't own. | |
| Exploiting time-sensitive vulnerabilities | Practitioner | Time two reset requests to generate identical (predictable) tokens. | |
| Partial construction race conditions | Expert | Inject `param[]=` to match an uninitialized field during construction. | |
