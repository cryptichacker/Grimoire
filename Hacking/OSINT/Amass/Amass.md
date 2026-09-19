---
tags: [hacking, recon, amass, osint, subdomains]
type: cheatsheet
source: compiled reference (OWASP Amass)
last-verified: 2026-08-30
---

# Amass

## Up
- [[OSINT]]

OWASP Amass is an in-depth **attack-surface mapping and subdomain enumeration** tool. It combines dozens of passive data sources (certs, APIs, datasets), active DNS resolution/brute-forcing, and DNS/graph analysis to build the fullest possible map of an organization's external footprint. Authorized targets only.

---

## Subcommands

| Command | Purpose |
|---|---|
| `amass enum` | Discover subdomains (passive and/or active) |
| `amass intel` | Collect intel — orgs, ASNs, root domains, IP ranges |
| `amass viz` | Visualize results (graph, D3, etc.) |
| `amass track` | Compare enumerations over time (what changed) |
| `amass db` | Query the local results graph database |

---

## Enumeration

```bash
# passive only (no direct DNS to target — quiet)
amass enum -passive -d example.com

# active (adds DNS resolution + brute-force + permutations)
amass enum -active -d example.com

# multiple domains / from a file
amass enum -d example.com,example.org
amass enum -df domains.txt

# brute-force with a wordlist + more resolvers
amass enum -active -brute -w subdomains.txt -d example.com

# limit noise / control speed
amass enum -passive -d example.com -timeout 15
```

Output each found name and save:

```bash
amass enum -passive -d example.com -o subs.txt          # plain output file
amass enum -d example.com -json out.json                # JSON records
```

---

## Intel (find what to even enumerate)

```bash
amass intel -d example.com                       # related root domains
amass intel -org "Example Inc"                   # search by org name → ASNs
amass intel -asn 15169                           # domains within an ASN
amass intel -cidr 192.0.2.0/24                   # reverse-DNS a range
amass intel -whois -d example.com                # reverse WHOIS for related domains
```

`intel` runs **before** `enum`: discover the org's root domains/IP ranges, then enumerate each.

---

## Data Sources & Config

```bash
amass enum -list                                 # list available data sources
# config file adds API keys for more sources (Shodan, Censys, VirusTotal, SecurityTrails, etc.)
# default config: ~/.config/amass/config.ini  (or -config path)
amass enum -config config.ini -d example.com
```

More API keys = far more passive coverage. Keys are optional but dramatically improve results.

---

## Visualization & Tracking

```bash
amass viz -d3 -o graph.html        # interactive graph of the discovered surface
amass viz -dot -o graph.dot

amass track -d example.com         # diff current vs previous enum (new/removed assets)
amass db -names -d example.com     # dump stored names from the graph DB
```

`track` is great for **continuous monitoring** — re-run periodically to catch newly exposed subdomains.

---

## Typical Workflow

```bash
# 1. find the org's root domains & ranges
amass intel -org "Example Inc" -o roots.txt

# 2. deep passive enum across them
amass enum -passive -df roots.txt -o subs_passive.txt

# 3. active + brute to catch unlisted names
amass enum -active -brute -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
  -d example.com -o subs_active.txt

# 4. merge, resolve to live hosts, feed to port scanning
sort -u subs_*.txt | httpx -silent > live.txt
```

Then hand `live.txt` to [[Nmap]]/[[Masscan]] for port/service work.

---

## Amass vs Sublist3r

- **[[Sublist3r]]** — fast, purely passive, quick wins, lightweight.
- **Amass** — far more thorough (active + brute + graph + intel + tracking), but slower and heavier.

Use Sublist3r for a quick first pass; use Amass for comprehensive coverage and ongoing monitoring.

---

## Tips

- Start `-passive` (quiet, safe); escalate to `-active`/`-brute` only within an authorized active-testing window.
- Populate API keys in the config — it's the biggest lever on result quality.
- Combine with certificate transparency (crt.sh) and other tools; no single source is complete.
- `amass track` turns recon into a repeatable, diffable process — ideal for bug bounty scope monitoring.
