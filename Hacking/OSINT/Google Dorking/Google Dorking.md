---
tags: [hacking, recon, osint, google-dorking]
type: cheatsheet
source: compiled reference (Google dorking / search operators)
last-verified: 2026-08-30
---

# Google Dorking

## Up
- [[OSINT]]

Google Dorking (Google Hacking) uses **advanced search operators** to surface information that's public but not obviously visible — exposed files, login portals, directory listings, config files, and indexed data. It's purely passive (you're querying a search engine, not the target) and is a fast way to find low-hanging exposure. Use findings responsibly and only act within authorized scope; don't access data you aren't permitted to.

---

## Core Operators

| Operator | Finds | Example |
|---|---|---|
| `site:` | Results from one domain | `site:example.com` |
| `inurl:` | Term in the URL | `inurl:admin` |
| `intitle:` | Term in the page title | `intitle:"index of"` |
| `intext:` | Term in the body | `intext:"password"` |
| `filetype:` / `ext:` | Specific file type | `filetype:pdf` |
| `cache:` | Google's cached copy | `cache:example.com` |
| `link:` | Pages linking to a URL | `link:example.com` |
| `related:` | Similar sites | `related:example.com` |
| `"..."` | Exact phrase | `"internal use only"` |
| `-` | Exclude a term | `site:example.com -www` |
| `OR` / `\|` | Either term | `inurl:login OR inurl:signin` |
| `*` | Wildcard word | `"example * admin"` |
| `AROUND(n)` | Terms within n words | `password AROUND(3) admin` |

Combine freely: `site:example.com filetype:pdf intext:confidential`

---

## Common Recon Dorks

```text
# subdomains / scope
site:*.example.com -www

# directory listings
intitle:"index of" site:example.com
intitle:"index of" "parent directory"

# login / admin panels
site:example.com inurl:(login | admin | dashboard | portal)

# exposed documents
site:example.com filetype:(pdf | xlsx | docx | csv)
site:example.com ext:(conf | cnf | ini | env | log | bak | sql)

# config / secrets patterns (exposure discovery)
intext:"DB_PASSWORD" filetype:env
intitle:"index of" ".git"
inurl:wp-config.php

# error messages / info leak
intext:"sql syntax near" | intext:"Warning: mysql_"
intitle:"phpinfo()" "PHP Version"

# cloud storage exposure
site:s3.amazonaws.com example
site:blob.core.windows.net example

# public code / docs leakage
site:pastebin.com example.com
site:github.com "example.com" password
```

---

## Beyond Google

The same idea works on other engines and specialized indexes:

| Engine | Notes |
|---|---|
| **Bing** | Supports `site:`, `ip:` (find sites on an IP), `filetype:` |
| **DuckDuckGo** | `site:`, `filetype:`, bangs (`!`) |
| **GitHub code search** | `org:acme password`, `filename:.env`, secrets in repos |
| **[[Shodan]]** | Dorking for devices/banners rather than pages |
| **Google Hacking Database (GHDB)** | Exploit-DB's curated dork catalog |

---

## Defensive / Blue-Team Angle

- Regularly dork **your own** domains to find accidental exposure before others do.
- Use `robots.txt`, proper auth, and storage ACLs — but remember `robots.txt` only *requests* non-indexing; it doesn't protect data (and even advertises paths).
- Request removal of sensitive indexed content via the search engine's removal tools, then fix the root exposure.
- Monitor Pastebin/GitHub for leaked credentials mentioning your domains.

---

## Tips

- Passive and legal to *search* — but **accessing** exposed systems/data you aren't authorized for is not. Report, don't exploit, outside scope.
- Chain `site:` narrowing with `filetype:`/`inurl:`/`intext:` to cut noise fast.
- The **GHDB** (Google Hacking Database) is the go-to reference for categorized, ready-made dorks.
- Search engines rate-limit heavy automated dorking and may CAPTCHA — keep it manual/light or use official APIs.
