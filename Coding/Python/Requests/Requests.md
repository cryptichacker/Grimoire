---
tags: [coding, python, http, requests]
type: cheatsheet
source: compiled reference (requests)
last-verified: 2026-08-27
---

# Requests

## Up
- [[Python]]

`requests` is the de-facto HTTP client library for Python — a friendly wrapper over urllib3. Install: `pip install requests`. Import: `import requests`.

---

## Basic Requests

```python
import requests

r = requests.get("https://api.example.com/users")
r = requests.post("https://api.example.com/users", json={"name": "Ada"})
r = requests.put(url, data={"k": "v"})
r = requests.patch(url, json={"age": 31})
r = requests.delete(url)
r = requests.head(url); requests.options(url)
```

---

## Query Params, Headers, Body

```python
# query string: ?page=2&limit=10
r = requests.get(url, params={"page": 2, "limit": 10})

# headers
r = requests.get(url, headers={"Authorization": "Bearer TOKEN",
                                "Accept": "application/json"})

# JSON body (sets Content-Type: application/json)
r = requests.post(url, json={"name": "Ada", "age": 30})

# form-encoded body
r = requests.post(url, data={"field": "value"})

# raw body / file upload
r = requests.post(url, data=b"raw bytes")
r = requests.post(url, files={"file": open("photo.png", "rb")})
r = requests.post(url, files={"file": ("name.csv", open("f.csv","rb"), "text/csv")})
```

---

## Reading the Response

```python
r.status_code            # 200
r.ok                     # True if < 400
r.text                   # body as str
r.content                # body as bytes
r.json()                 # parse JSON → dict/list
r.headers                # response headers (dict-like, case-insensitive)
r.headers["Content-Type"]
r.url                    # final URL (after redirects)
r.encoding               # 'utf-8'
r.cookies                # RequestsCookieJar
r.history                # list of redirect responses
r.elapsed                # timedelta of round-trip
```

---

## Error Handling

```python
try:
    r = requests.get(url, timeout=10)
    r.raise_for_status()                 # raise HTTPError for 4xx/5xx
    data = r.json()
except requests.exceptions.Timeout:
    print("timed out")
except requests.exceptions.ConnectionError:
    print("connection failed")
except requests.exceptions.HTTPError as e:
    print("bad status:", e.response.status_code)
except requests.exceptions.RequestException as e:
    print("request failed:", e)          # base class for all
```

**Always pass `timeout=`** — by default requests waits forever.

---

## Authentication

```python
# HTTP Basic
r = requests.get(url, auth=("user", "pass"))

# Bearer token
r = requests.get(url, headers={"Authorization": "Bearer TOKEN"})

from requests.auth import HTTPBasicAuth, HTTPDigestAuth
requests.get(url, auth=HTTPDigestAuth("user", "pass"))
```

---

## Sessions (connection reuse + persistent state)

```python
s = requests.Session()
s.headers.update({"Authorization": "Bearer TOKEN"})
s.get("https://api.example.com/login")     # cookies persist across calls
r = s.get("https://api.example.com/profile")
s.close()

# as context manager
with requests.Session() as s:
    s.get(url)
```

A `Session` reuses the underlying TCP connection (faster) and carries cookies/headers/auth across requests — use it whenever you make more than one call to the same host.

---

## Timeouts, Retries, Redirects

```python
requests.get(url, timeout=5)               # total timeout (seconds)
requests.get(url, timeout=(3, 10))         # (connect, read)

requests.get(url, allow_redirects=False)   # don't follow 3xx
requests.get(url, verify=False)            # skip TLS verification (avoid in prod)
requests.get(url, proxies={"https": "http://proxy:8080"})
requests.get(url, stream=True)             # don't download body immediately

# robust retry policy
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry = Retry(total=3, backoff_factor=0.5,
              status_forcelist=[429, 500, 502, 503, 504])
s = requests.Session()
s.mount("https://", HTTPAdapter(max_retries=retry))
```

---

## Streaming & Large Downloads

```python
with requests.get(url, stream=True) as r:
    r.raise_for_status()
    with open("big.zip", "wb") as f:
        for chunk in r.iter_content(chunk_size=8192):
            f.write(chunk)

for line in requests.get(url, stream=True).iter_lines():
    print(line.decode())
```

---

## Tips

- `r.json()` raises `json.JSONDecodeError` on non-JSON bodies — guard it.
- Use `params=` (not string concatenation) so values are URL-encoded correctly.
- Set a descriptive `User-Agent` header when scraping APIs politely.
- For heavy async workloads consider `httpx` (drop-in-ish async API) or `aiohttp`.
- Never disable `verify` against real endpoints; fix the cert chain instead.
