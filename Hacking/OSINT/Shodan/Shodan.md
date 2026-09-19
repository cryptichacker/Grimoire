---
tags: [hacking, recon, shodan, osint]
type: cheatsheet
source: compiled reference (Shodan)
last-verified: 2026-08-30
---

# Shodan

## Up
- [[OSINT]]

Shodan is a search engine for **internet-connected devices**. Instead of indexing web page content, it continuously scans the internet and indexes **service banners** — open ports, protocols, software versions, TLS certs, and metadata for servers, routers, webcams, ICS/SCADA, databases, IoT, and more. Purely passive from your side (you query Shodan's existing data), making it a staple of early recon. Query only within authorized scope; don't interact with exposed devices you find.

---

## Web vs CLI vs API

- **Web** — https://www.shodan.io (search bar + filters + maps + reports).
- **CLI** — `pip install shodan` then `shodan init <API_KEY>`.
- **API** — Python (`import shodan`), REST; automate lookups and monitoring.

```bash
pip install shodan
shodan init YOUR_API_KEY
shodan info                       # show plan / query credits
```

---

## Search Filters (web & API)

| Filter | Finds |
|---|---|
| `hostname:example.com` | Hosts in a domain |
| `net:192.0.2.0/24` | Hosts in an IP range |
| `org:"Example Inc"` | Assets owned by an org |
| `asn:AS15169` | Hosts in an ASN |
| `port:22` | Specific open port |
| `product:nginx` / `product:Apache` | Software |
| `version:"1.18.0"` | Software version |
| `os:"Windows"` | Operating system |
| `country:US` / `city:"Austin"` | Geolocation |
| `ssl:"example.com"` | Cert common-name/SAN match |
| `ssl.cert.expired:true` | Expired TLS certs |
| `http.title:"Login"` | Web page title |
| `http.status:200` | HTTP status |
| `vuln:CVE-2021-44228` | Hosts flagged for a CVE (paid) |
| `has_screenshot:true` | Devices with a screenshot |

Combine with quotes and multiple filters:
`org:"Example Inc" port:3389 os:"Windows"`

---

## CLI Commands

```bash
shodan search --fields ip_str,port,org 'apache country:US'   # search
shodan count 'port:22 country:DE'          # just the result count (no credits for count)
shodan host 8.8.8.8                         # everything known about an IP
shodan domain example.com                   # subdomains + DNS + open ports
shodan scan submit 192.0.2.10               # request an on-demand scan (paid, authorized only)
shodan alert create "prod" 192.0.2.0/24     # monitor a range for changes
shodan alert list
shodan stats --facet port 'org:"Example Inc"'   # aggregate stats/facets
shodan myip                                 # your public IP
```

---

## Python API

```python
import shodan
api = shodan.Shodan("YOUR_API_KEY")

# host lookup
host = api.host("8.8.8.8")
print(host["ip_str"], host.get("org"), host.get("ports"))
for item in host["data"]:
    print(item["port"], item["transport"], item.get("product"))

# search
res = api.search('org:"Example Inc" port:443')
print("total:", res["total"])
for r in res["matches"][:10]:
    print(r["ip_str"], r["port"], r.get("http", {}).get("title"))
```

---

## What to Look For in Recon

- Exposed admin/management ports: **RDP 3389, SSH 22, SMB 445, databases (3306/5432/27017/6379), Kubernetes 6443/10250, Docker 2375**.
- Default banners / outdated versions → known CVEs.
- Expired or mismatched TLS certs, self-signed on prod.
- Forgotten dev/staging hosts, panels (`http.title:"Login"`), and services that shouldn't be public.
- Cert SANs (`ssl:`) often reveal additional subdomains/hostnames.

---

## Related Engines

| Engine | Notes |
|---|---|
| **Shodan** | Largest banner index, great filters, alerts |
| **Censys** | Strong TLS/cert + host data, good API |
| **FOFA / ZoomEye** | Similar, strong in some regions |
| **BinaryEdge** | Another internet-scan dataset |

Cross-check across engines — coverage and freshness differ.

---

## Tips

- **Passive** for you — you're querying Shodan's existing scans, not touching targets. Actually *connecting* to a device you discover is active and must be in scope.
- Free accounts have limited filters/credits; `shodan count` is cheap for sizing.
- Use `shodan alert`/monitor for continuous exposure detection on your own ranges (defensive use is excellent here).
- Pair with [[Amass]]/[[theHarvester]] (which can consume Shodan via API key) for a fuller external picture.
