---
tags: [hacking, recon, sublist3r, osint, subdomains]
type: cheatsheet
source: compiled reference (Sublist3r)
last-verified: 2026-08-30
---

# Sublist3r

## Up
- [[OSINT]]

Sublist3r is a lightweight Python tool for **fast, passive subdomain enumeration**. It queries public search engines and passive DNS sources (Google, Bing, Yahoo, Baidu, VirusTotal, ThreatCrowd, DNSdumpster, crt.sh, etc.) and can optionally brute-force via its bundled **subbrute** engine. Great for a quick first pass; use [[Amass]] for depth. Authorized scope only.

---

## Install & Basic Use

```bash
pip install sublist3r
# or: git clone https://github.com/aboul3la/Sublist3r && pip install -r requirements.txt

sublist3r -d example.com                   # enumerate subdomains
python sublist3r.py -d example.com         # from a clone
```

---

## Common Options

```bash
sublist3r -d example.com -o subs.txt       # save results to a file
sublist3r -d example.com -v                # verbose (show results in real time)
sublist3r -d example.com -t 50             # threads for brute-force
sublist3r -d example.com -b                # enable subbrute brute-forcing
sublist3r -d example.com -e google,bing    # limit to specific engines
sublist3r -d example.com -p 80,443         # show subdomains with these ports open
```

| Flag | Meaning |
|---|---|
| `-d` | Target domain (required) |
| `-o` | Output file |
| `-b` | Enable brute-force (subbrute) |
| `-e` | Comma-separated engines to use |
| `-t` | Threads (brute-force) |
| `-p` | Check these ports on found subdomains |
| `-v` | Verbose |

---

## Typical Workflow

```bash
# quick passive sweep, save output
sublist3r -d example.com -o subs.txt

# add brute-force for a bit more coverage
sublist3r -d example.com -b -t 100 -o subs_brute.txt

# merge, dedupe, resolve to live hosts
sort -u subs*.txt | httpx -silent > live.txt
```

Then feed live hosts to [[Nmap]]/[[Masscan]] and web tooling.

---

## Where It Fits

| Tool | Strength |
|---|---|
| **Sublist3r** | Fast, simple, passive; quick wins |
| **[[Amass]]** | Comprehensive: active + brute + graph + intel + tracking |
| **crt.sh / cert transparency** | Subdomains from issued TLS certs |
| **[[theHarvester]]** | Also pulls emails/hosts/names, not just subdomains |

---

## Tips

- Mostly passive by default — low noise, safe for an early recon pass.
- Results are only as fresh as the public sources; **combine multiple tools** (Amass, crt.sh, theHarvester) — none is complete alone.
- Search engines may rate-limit/CAPTCHA heavy use; that's expected.
- The project is older and less actively maintained than Amass — keep it as a quick supplementary tool, not your only source.
