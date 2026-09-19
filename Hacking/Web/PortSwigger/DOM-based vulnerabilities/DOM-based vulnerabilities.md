---
tags: [hacking, web, portswigger, dom-based]
type: vulnerability-category
source: https://portswigger.net/web-security/dom-based
last-verified: 2026-08-25
---

# DOM-based vulnerabilities

## Up
- [[PortSwigger]]

## What it is
DOM-based vulnerabilities arise entirely in client-side JavaScript: a **source** (attacker-controllable data) flows into a **sink** (a dangerous function/property) without safe handling — a client-side **taint flow**. The server may never see the payload (e.g. it's in the URL fragment).

## Sources and sinks
**Common sources:** `location` (`location.search`, `location.hash`), `document.URL`, `document.referrer`, `document.cookie`, `window.name`, `postMessage` web messages, `localStorage`/`sessionStorage`.

**Common sinks (by vuln):**

| Type | Example sink |
|---|---|
| DOM XSS | `innerHTML`, `document.write()`, `eval()` |
| Open redirection | `location` / `location.href` |
| Cookie manipulation | `document.cookie` |
| JavaScript injection | `eval()`, `Function()` |
| Document-domain manipulation | `document.domain` |
| WebSocket-URL poisoning | `new WebSocket()` |
| Link manipulation | `element.src` / `href` |
| Web-message manipulation | `postMessage()` handling |
| Ajax request-header manipulation | `setRequestHeader()` |
| Local file-path manipulation | `FileReader.readAsText()` |
| Client-side SQL injection | `ExecuteSql()` |
| HTML5 storage manipulation | `sessionStorage.setItem()` |
| Client-side XPath injection | `document.evaluate()` |
| Client-side JSON injection | `JSON.parse()` |
| DOM-data manipulation | `element.setAttribute()` |
| Denial of service | `RegExp()` |

## Examples
**DOM open redirection** — a hash-controlled redirect:
```javascript
goto = location.hash.slice(1)
if (goto.startsWith('https:')) { location = goto; }
```
`https://innocent.com/example#https://evil-user.net` redirects to the attacker.

**Web-message manipulation** — a page adds a `message` event listener and uses `event.data` in a sink without checking `event.origin`; any site that frames it can `postMessage` a payload:
```javascript
window.addEventListener('message', e => { document.body.innerHTML = e.data; });
```

**DOM clobbering** — when script has no obvious sink, inject HTML whose `id`/`name` clobbers a global the code relies on, e.g.:
```html
<a id="x"><a id="x" name="y" href="javascript:alert(1)">
```
making `x.y` resolve to attacker-controlled markup used unsafely later.

## Prevention
- Don't let untrusted data reach a dynamic sink; prefer safe APIs (`textContent`, `setAttribute` with validation).
- For web messages, **verify `event.origin`** and validate `event.data`.
- Apply context-appropriate encoding/validation (HTML, JS, URL) and allow-listing.
- Consider Trusted Types to lock down DOM-XSS sinks.

## Labs
7 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| DOM XSS using web messages | Apprentice | `postMessage` a payload into an unchecked `innerHTML` handler. | |
| DOM XSS using web messages and a JavaScript URL | Practitioner | Deliver a `javascript:` URL through the web-message sink. | |
| DOM XSS using web messages and JSON.parse | Practitioner | Send a JSON message parsed then written to a sink. | |
| DOM-based open redirection | Apprentice | Control the redirect target via the URL/hash source. | |
| DOM-based cookie manipulation | Apprentice | Inject into `document.cookie` reflected into the page. | |
| Exploiting DOM clobbering to enable XSS | Practitioner | Clobber a global with anchor elements to reach a sink. | |
| Clobbering DOM attributes to bypass HTML filters | Expert | Use clobbering to defeat a sanitizer and land XSS. | |
