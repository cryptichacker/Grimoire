---
tags: [hacking, recon, theharvester, osint]
type: cheatsheet
source: compiled reference (theHarvester)
last-verified: 2026-08-30
---

# theHarvester

## Up
- [[OSINT]]

theHarvester is a passive OSINT tool that gathers **emails, employee names, subdomains, hostnames, IPs, and URLs** for a domain from many public sources (search engines, certificate transparency, PGP key servers, Shodan, and specialized data providers). Classic early-stage footprinting for building a target profile and email lists (e.g. for password-spray scope). Authorized engagements only.

---

## Basic Usage

```bash
theHarvester -d example.com -b all             # all sources
theHarvester -d example.com -b bing
theHarvester -d example.com -b crtsh,duckduckgo,otx
theHarvester -d example.com -l 500 -b google   # limit to 500 results
```

| Flag | Meaning |
|---|---|
| `-d` | Target domain/company |
| `-b` | Data source(s) — comma-separated, or `all` |
| `-l` | Limit number of results |
| `-S` | Start result offset |
| `-g` | Use Google dorking mode |
| `-s` | Use Shodan for discovered hosts |
| `-f` | Output to file (HTML/JSON/XML depending on version) |
| `-n` | DNS reverse lookup on found ranges |
| `-c` | DNS brute-force |

---

## Data Sources (`-b`)

Common sources include `bing`, `duckduckgo`, `google`, `crtsh` (cert transparency), `otx` (AlienVault), `hackertarget`, `threatcrowd`, `urlscan`, `rapiddns`, `subdomaincenter`, `shodan`, `censys`, `github-code`, `pgp`, `virustotal`. Some need API keys configured (see below); `-b all` uses everything available.

```bash
theHarvester -d example.com -b all -f results
# outputs results.json / results.xml (version-dependent) + on-screen summary
```

---

## API Keys

Some sources (Shodan, Censys, GitHub, SecurityTrails, Hunter, etc.) require keys, set in the config:

```
# ~/.theHarvester/api-keys.yaml   (or /etc/theHarvester/api-keys.yaml)
apikeys:
  shodan:
    key: YOUR_KEY
  github:
    key: YOUR_KEY
```

More keys → substantially better coverage, same as [[Amass]].

---

## What You Get & Why It Matters

| Output | Use |
|---|---|
| **Emails** | Build username lists, derive email format (`f.last@`), phishing-scope/awareness testing, password-spray targets |
| **Names** | Map employees → usernames; org-chart building |
| **Subdomains / hosts** | Expand attack surface (feed to [[Nmap]]) |
| **IPs / ranges** | Netblock discovery |
| **URLs** | Interesting endpoints, old pages |

---

## Typical Workflow

```bash
# broad passive gather with output
theHarvester -d example.com -b all -f harvest

# extract just emails for later use
grep -Eio '[a-z0-9._%+-]+@example\.com' harvest.json | sort -u > emails.txt

# extract subdomains → resolve → port scan
# (combine with Amass/Sublist3r/crt.sh, none is complete alone)
```

---

## Tips

- Fully passive — safe and quiet for early recon; it never touches the target directly (only third-party data).
- **Cross-reference** with LinkedIn/social to confirm the email naming convention.
- Combine subdomain output with [[Amass]] + [[Sublist3r]] + crt.sh for the widest coverage.
- Handle discovered PII responsibly and only within the rules of engagement.
