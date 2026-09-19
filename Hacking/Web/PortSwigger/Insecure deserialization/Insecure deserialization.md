---
tags: [hacking, web, portswigger, insecure-deserialization]
type: vulnerability-category
source: https://portswigger.net/web-security/deserialization
last-verified: 2026-08-25
---

# Insecure deserialization

## Up
- [[PortSwigger]]

## What it is
**Serialization** turns an object into a byte/string stream (preserving attributes and values) for storage or transport; **deserialization** rebuilds the object. **Insecure deserialization** is deserializing user-controllable data, letting an attacker tamper with the serialized object — and because deserialization can instantiate arbitrary available classes and run code during the process, the damage often happens *before* deserialization even completes ("object injection").

## Why it arises
- Developers assume post-deserialization validation is enough.
- Binary formats create false confidence that users can't tamper with them.
- Huge class/dependency pools create unpredictable, exploitable code paths.

## Impact
A big jump in attack surface: remote code execution, privilege escalation, arbitrary file access, and DoS.

## Identifying it
- **PHP** (human-readable): `O:8:"ClassName":2:{s:4:"name";s:6:"carlos";s:7:"isAdmin";b:0;}`.
- **Java**: binary objects start with `ac ed 00 05` (hex) or `rO0AB...` (Base64).
Also look for cookies/params holding serialized blobs.

## Exploitation
- **Modifying attributes** — flip `isAdmin` from `b:0` to `b:1`; adjust the string-length prefixes when you change values (`s:6:"carlos"` → `s:5:"admin"`).
- **Modifying data types** — abuse loose comparison (PHP `==`): change a string to an integer `0` or a boolean so a password/token check passes.
- **Magic methods** — code in methods auto-invoked during the object lifecycle (`__wakeup()`, `__destruct()` in PHP; `readObject()` in Java) runs on your crafted object.
- **Gadget chains** — chain existing classes' magic methods to reach a dangerous sink (RCE). Use pre-built chains: **ysoserial** (Java), **PHPGGC** (PHP).
- **PHAR deserialization** — PHP file operations on a `phar://` path deserialize the archive's metadata, triggering a chain without any explicit `unserialize()`.

## Prevention
- **Don't deserialize untrusted input** — the only robust defense.
- If unavoidable: verify a **digital signature/HMAC** over the data *before* deserializing; use per-class/format-limited deserialization; run in low-privilege, sandboxed context. Don't rely on blocklisting gadget classes.

## Labs
10 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Modifying serialized objects | Apprentice | Flip `admin` boolean in a PHP-serialized cookie. | |
| Modifying serialized data types | Practitioner | Abuse PHP loose comparison by changing types. | |
| Using application functionality to exploit insecure deserialization | Practitioner | Point a filepath attribute at a file to delete via `__destruct`. | |
| Arbitrary object injection in PHP | Practitioner | Inject a crafted object of a class with a useful magic method. | |
| Exploiting Java deserialization with Apache Commons | Practitioner | ysoserial CommonsCollections chain for RCE. | |
| Exploiting PHP deserialization with a pre-built gadget chain | Practitioner | PHPGGC to build the serialized RCE payload; sign it if needed. | |
| Exploiting Ruby deserialization using a documented gadget chain | Practitioner | Use a public Ruby gadget chain. | |
| Developing a custom gadget chain for Java deserialization | Expert | Build a bespoke chain from app classes. | |
| Developing a custom gadget chain for PHP deserialization | Expert | Build a bespoke PHP chain. | |
| Using PHAR deserialization to deploy a custom gadget chain | Expert | Upload a polyglot PHAR, trigger via `phar://`. | |
