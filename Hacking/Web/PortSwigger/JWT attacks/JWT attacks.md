---
tags: [hacking, web, portswigger, jwt]
type: vulnerability-category
source: https://portswigger.net/web-security/jwt
last-verified: 2026-08-25
---

# JWT attacks

## Up
- [[PortSwigger]]

## What it is
A JWT is `header.payload.signature`, each part base64url-encoded. The header names the algorithm; the payload holds claims (`sub`, `role`, `exp`…); the signature protects both. **JWS** = signed, **JWE** = encrypted (usually JWS is meant). Attacks come from servers that **don't verify the signature properly**, letting an attacker tamper with claims (e.g. `role`, `sub`).

## Flawed signature verification
- **Not checking at all** — code calls `decode()` instead of `verify()`; change the payload freely.
- **`alg: none`** — set the header algorithm to `none` and drop the signature (keep the trailing dot). Bypass naive filters with case/encoding tricks (`nOnE`).
  ```json
  {"alg":"none","typ":"JWT"}
  ```

## Weak signing key (HS256)
Symmetric HMAC keys are just strings; developers leave defaults/weak secrets. Brute-force offline:
```
hashcat -a 0 -m 16500 <jwt> wordlist.txt
```
Then re-sign your tampered token with the cracked secret.

## Header parameter injection
Three header params can point at attacker-chosen keys:
- **`jwk`** — an embedded public key. If the server trusts it, generate your own keypair, sign with your private key, embed the matching public key in `jwk`.
- **`jku`** — a URL the server fetches keys from; host your JWKS and (often via URL-parsing tricks) get it accepted.
- **`kid`** — key identifier used to select the key. Abuse it for **path traversal** (`"kid":"../../dev/null"` → sign with an empty string) or **SQL injection** if keys are DB-backed.

## Algorithm confusion (RS256 → HS256)
If the server verifies RS256 but blindly uses the algorithm from the token, forge an HS256 token signed with the server's **public key** used as the HMAC secret:
1. Obtain the public key (JWKS endpoint / certificate / derive from two tokens).
2. Set `"alg":"HS256"`, tamper the payload.
3. HMAC-sign using the public key (PEM) as the secret.
The server verifies with its public key as an HMAC key and accepts it.

## Prevention
- Use a vetted library; verify the signature and **enforce an expected algorithm** (reject `none` and unexpected algs).
- Whitelist `jku` hosts; validate `kid` against traversal/SQLi; don't trust embedded `jwk`.
- Set `exp`, include an `aud` claim, and support revocation. Use strong secrets.

## Labs
8 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| JWT authentication bypass via unverified signature | Apprentice | Change `sub` to `administrator`; signature unchecked. | |
| JWT authentication bypass via flawed signature verification | Apprentice | Set `alg:none`, strip the signature. | |
| JWT authentication bypass via weak signing key | Practitioner | hashcat-crack the HS256 secret, re-sign. | |
| JWT authentication bypass via jwk header injection | Practitioner | Embed your own public key in `jwk`. | |
| JWT authentication bypass via jku header injection | Practitioner | Host a JWKS and point `jku` at it. | |
| JWT authentication bypass via kid header path traversal | Practitioner | `kid` → `/dev/null`, sign with empty key. | |
| JWT authentication bypass via algorithm confusion | Expert | RS256→HS256 using the exposed public key. | |
| JWT authentication bypass via algorithm confusion with no exposed key | Expert | Derive the public key from two tokens, then confuse. | |
