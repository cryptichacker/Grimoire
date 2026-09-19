---
tags: [hacking, recon, nmap, scanning]
type: cheatsheet
source: compiled reference (Nmap)
last-verified: 2026-08-30
---

# Nmap

## Up
- [[Reconnaissance]]

Nmap ("Network Mapper") is the standard tool for host discovery, port scanning, service/version detection, OS fingerprinting, and scriptable enumeration via the **NSE** (Nmap Scripting Engine). Only scan systems you're authorized to test.

---

## Target Specification

```bash
nmap 10.0.0.1                      # single host
nmap 10.0.0.1 10.0.0.2             # multiple
nmap 10.0.0.0/24                   # CIDR range
nmap 10.0.0.1-50                   # octet range
nmap scanme.nmap.org               # hostname
nmap -iL targets.txt               # from a file
nmap 10.0.0.0/24 --exclude 10.0.0.5
```

---

## Host Discovery (find live hosts)

```bash
nmap -sn 10.0.0.0/24               # ping sweep, NO port scan
nmap -Pn 10.0.0.1                  # skip discovery, treat as up (bypass ICMP block)
nmap -PS22,80,443 10.0.0.0/24      # TCP SYN ping to specific ports
nmap -PU53 10.0.0.0/24             # UDP ping
nmap -n 10.0.0.1                   # never do DNS resolution (faster)
```

---

## Scan Types

| Flag | Scan | Notes |
|---|---|---|
| `-sS` | TCP SYN ("half-open") | Default when root; fast, stealthier |
| `-sT` | TCP connect | Full handshake; used when non-root |
| `-sU` | UDP | Slow; needed for DNS/SNMP/DHCP etc. |
| `-sA` | TCP ACK | Firewall rule mapping (filtered vs unfiltered) |
| `-sN`/`-sF`/`-sX` | Null / FIN / Xmas | Stealth/firewall-evasion probes |
| `-sV` | Service/version detection | Grabs banners, identifies software |
| `-O` | OS detection | Fingerprints the OS |
| `-sC` | Default NSE scripts | Equivalent to `--script=default` |
| `-A` | Aggressive | `-sV -O -sC --traceroute` combined |

```bash
sudo nmap -sS 10.0.0.1             # SYN scan (needs root)
nmap -sV 10.0.0.1                  # version detection
sudo nmap -sU --top-ports 20 10.0.0.1   # top 20 UDP ports
```

---

## Port Selection

```bash
nmap -p 80,443 10.0.0.1            # specific ports
nmap -p 1-1000 10.0.0.1           # range
nmap -p- 10.0.0.1                 # ALL 65535 ports
nmap -F 10.0.0.1                  # fast: top 100 ports
nmap --top-ports 1000 10.0.0.1
nmap -p U:53,T:80,443 10.0.0.1    # mixed TCP/UDP
```

---

## Timing & Performance

```bash
nmap -T4 10.0.0.1                 # timing template 0-5 (0=paranoid, 5=insane)
nmap --min-rate 1000 10.0.0.1     # send ≥1000 packets/sec
nmap --max-retries 2 10.0.0.1
nmap --host-timeout 30m 10.0.0.1
```

`-T4` is the common default for a lab/internal engagement; lower it (`-T2`) to be quieter, raise (`-T5`) only on fast reliable networks.

---

## The Recommended Combos

```bash
# 1. fast full-port TCP sweep to find open ports
sudo nmap -sS -p- --min-rate 5000 -T4 10.0.0.1 -oA allports

# 2. deep scan of ONLY the open ports found
nmap -sV -sC -p 22,80,443 10.0.0.1 -oA deep

# aggressive one-shot for a single box
sudo nmap -A -T4 10.0.0.1

# UDP top ports (slow but important)
sudo nmap -sU --top-ports 100 -T4 10.0.0.1
```

---

## NSE — Nmap Scripting Engine

```bash
nmap --script=default 10.0.0.1              # or -sC
nmap --script=vuln 10.0.0.1                 # known-vuln checks
nmap --script=http-enum -p80 10.0.0.1       # web content discovery
nmap --script=smb-enum-shares,smb-os-discovery -p445 10.0.0.1
nmap --script "http-*" -p80 10.0.0.1        # wildcard category
nmap --script-help=ssl-heartbleed
```

Script categories: `auth`, `default`, `discovery`, `safe`, `intrusive`, `vuln`, `exploit`, `brute`, `malware`. Scripts live in `/usr/share/nmap/scripts/`.

---

## Output Formats

```bash
nmap -oN out.txt 10.0.0.1          # normal (human-readable)
nmap -oG out.grep 10.0.0.1         # grepable
nmap -oX out.xml 10.0.0.1          # XML (for tools/reports)
nmap -oA basename 10.0.0.1         # all three at once (best practice)
nmap -v 10.0.0.1                   # verbose (-vv for more)
nmap --reason 10.0.0.1             # why a port is in its state
```

Convert XML → HTML report with `xsltproc out.xml -o report.html`.

---

## Firewall / IDS Evasion (concepts)

```bash
nmap -f 10.0.0.1                   # fragment packets
nmap -D RND:10 10.0.0.1            # decoy scan (mix in fake source IPs)
nmap -S <spoofed_ip> 10.0.0.1      # spoof source (needs -e, rarely useful)
nmap --source-port 53 10.0.0.1     # scan from a trusted port
nmap --data-length 25 10.0.0.1     # pad packets
nmap -T1 10.0.0.1                  # slow to stay under thresholds
```

---

## Port States

| State | Meaning |
|---|---|
| **open** | An application is accepting connections |
| **closed** | Reachable but no app listening |
| **filtered** | A firewall is blocking probes (no response) |
| **unfiltered** | Reachable but state undetermined (ACK scan) |
| **open\|filtered** | Can't tell (common in UDP) |

---

## Tips

- Root/`sudo` unlocks SYN (`-sS`), OS detection, and raw-packet features; without it Nmap falls back to `-sT`.
- Always `-oA` your scans — you'll want the data for reporting and re-parsing.
- Two-stage (fast port find → deep scan of open ports) is far quicker than `-sV -p-` on everything.
- Don't forget **UDP** — DNS, SNMP, TFTP, IKE hide there and are often overlooked.
- Pair with **[[Masscan]]** for very large ranges, then feed open ports into Nmap for detail.
