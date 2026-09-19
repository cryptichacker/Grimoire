---
tags: [hacking, recon, recon-ng, osint]
type: cheatsheet
source: compiled reference (recon-ng)
last-verified: 2026-08-30
---

# recon-ng

## Up
- [[OSINT]]

recon-ng is a full-featured **web reconnaissance framework** with a Metasploit-style interactive console. It organizes OSINT into **modules** (each pulls or transforms data), stores everything in a **workspace database**, and lets you chain modules so one module's output (e.g. domains) feeds the next (e.g. hosts, contacts, credentials-in-breaches). Authorized scope only.

---

## Launch & Workspaces

```bash
recon-ng                                  # start the console
recon-ng -w example                       # start in a named workspace

# inside the console:
workspaces list
workspaces create example
workspaces load example
workspaces remove old
```

Each workspace has its own database (domains, hosts, contacts, credentials, ports, etc.).

---

## Marketplace (modules)

Modules aren't all installed by default — install from the marketplace:

```
marketplace search           # list all modules
marketplace search hackertarget
marketplace info recon/domains-hosts/hackertarget
marketplace install recon/domains-hosts/hackertarget
marketplace install all      # install everything (some need API keys)
marketplace refresh
```

Module path format: `recon/<input>-<output>/<name>`, e.g. `recon/domains-hosts/…` takes domains → produces hosts.

---

## Using a Module

```
modules load recon/domains-hosts/hackertarget
info                         # show options + description
options set SOURCE example.com
run

modules load recon/domains-contacts/whois_pocs
run

back                         # unload module
modules search subdomain     # find loaded/installed modules
```

Set `SOURCE` to a literal value, or leave it as `default` to pull from the workspace DB (e.g. all stored domains).

---

## The Database (workspace data)

```
db schema                    # tables: domains, companies, hosts, contacts, credentials, ports, ...
db insert domains            # manually add a seed domain
show domains                 # view stored rows
show hosts
show contacts
db query SELECT * FROM hosts WHERE ip_address IS NOT NULL
```

Modules read from and write back to these tables, which is how chaining works: add a domain → run a domains-hosts module → hosts table fills → run a hosts-ports module, etc.

---

## API Keys

```
keys list
keys add shodan_api YOUR_KEY
keys add virustotal_api YOUR_KEY
keys remove shodan_api
```

Many high-value modules (Shodan, VirusTotal, BuiltWith, Hunter, SecurityTrails) need keys.

---

## Example Chain (domain → attack surface)

```
workspaces create acme
db insert domains            # -> enter: example.com

# subdomains / hosts
modules load recon/domains-hosts/hackertarget
run
modules load recon/domains-hosts/certificate_transparency
run

# resolve + geolocate
modules load recon/hosts-hosts/resolve
run

# contacts / emails
modules load recon/domains-contacts/whois_pocs
run

# report
modules load reporting/html
options set FILENAME /tmp/acme_recon.html
run
```

Reporting modules (`reporting/html`, `reporting/csv`, `reporting/json`, `reporting/list`) export the whole workspace.

---

## Tips

- Think in **module chains**: pick modules whose input matches data you already have and whose output you want next.
- `default` sources pull from the DB automatically — populate `domains` first, then run domain-input modules across all of them at once.
- Marketplace modules break/appear over time; `marketplace refresh` and check `info` for required keys/dependencies.
- Console UX mirrors Metasploit (`modules load`, `options set`, `run`, `back`) — familiar if you know msfconsole.
- Great as an **aggregator/orchestrator** that ties [[Shodan]], cert transparency, and WHOIS data into one queryable workspace + report.
