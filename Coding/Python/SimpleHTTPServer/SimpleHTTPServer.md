---
tags: [coding, python, http, server]
type: cheatsheet
source: compiled reference (Python 3 http.server)
last-verified: 2026-08-27
---

# SimpleHTTPServer

## Up
- [[Python]]

Python's built-in one-command static file server. In Python 3 the old `SimpleHTTPServer` module became `http.server`. It's ideal for quickly sharing a directory, previewing a static site, or testing front-end code against a real HTTP origin (avoids `file://` CORS issues).

---

## Quick Start (CLI)

```bash
python3 -m http.server                 # serve CWD on http://0.0.0.0:8000
python3 -m http.server 9000            # custom port
python3 -m http.server 8000 --bind 127.0.0.1   # localhost only
python3 -m http.server --directory /path/to/dir 8000   # serve another dir (3.7+)
python3 -m http.server --cgi 8000      # enable CGI handler
```

Stop with `Ctrl-C`. It auto-generates a directory listing when no `index.html` is present, and serves `index.html` automatically when it is.

---

## Programmatic Use

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler

server = HTTPServer(("0.0.0.0", 8000), SimpleHTTPRequestHandler)
print("Serving on http://localhost:8000")
server.serve_forever()          # blocks; Ctrl-C to stop
```

```python
# Serve a specific directory (3.7+)
from functools import partial
from http.server import HTTPServer, SimpleHTTPRequestHandler

handler = partial(SimpleHTTPRequestHandler, directory="/var/www")
HTTPServer(("", 8000), handler).serve_forever()
```

---

## Custom Request Handler

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json

class MyHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.end_headers()
            self.wfile.write(json.dumps({"status": "ok"}).encode())
        else:
            self.send_error(404, "Not Found")

    def do_POST(self):
        length = int(self.headers.get("Content-Length", 0))
        body = self.rfile.read(length)             # raw request body
        data = json.loads(body or b"{}")
        self.send_response(201)
        self.send_header("Content-Type", "application/json")
        self.end_headers()
        self.wfile.write(json.dumps({"received": data}).encode())

    def log_message(self, fmt, *args):             # silence/redirect logging
        pass

HTTPServer(("", 8000), MyHandler).serve_forever()
```

| Method / attr | Purpose |
|---|---|
| `do_GET`, `do_POST`, `do_PUT`, `do_DELETE` | Handle each HTTP verb |
| `self.path` | Request path + query string |
| `self.headers` | Request headers (a `Message` object) |
| `self.rfile` / `self.wfile` | Read request body / write response body (bytes) |
| `send_response(code)` | Status line + default headers |
| `send_header(k, v)` / `end_headers()` | Add headers, then terminate the header block |
| `send_error(code, msg)` | Convenience error response |

---

## Threaded Server (handle concurrent requests)

```python
from http.server import SimpleHTTPRequestHandler, ThreadingHTTPServer

# ThreadingHTTPServer (3.7+) serves each request in its own thread
ThreadingHTTPServer(("", 8000), SimpleHTTPRequestHandler).serve_forever()
```

The plain `HTTPServer` is single-threaded — one slow request blocks the rest. Use `ThreadingHTTPServer` for anything beyond trivial local testing.

---

## Notes & Gotchas

- **Not for production.** No auth, no HTTPS, minimal hardening — it's a dev/utility tool. Use nginx, gunicorn+Flask/FastAPI, or a CDN for real serving.
- Binds to `0.0.0.0` by default, exposing the folder to your whole LAN. Use `--bind 127.0.0.1` to keep it local.
- Serves the current working directory — be careful not to expose secrets/parent files.
- All response bodies must be `bytes` (`.encode()` your strings).
- For a quick HTTPS test server you must wrap the socket with `ssl.SSLContext` manually; `http.server` has no built-in TLS flag.
