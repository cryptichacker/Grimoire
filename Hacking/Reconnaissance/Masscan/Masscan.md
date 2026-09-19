---
tags: [hacking, recon, masscan, scanning]
type: cheatsheet
source: compiled reference (masscan)
last-verified: 2026-08-30
---

# Masscan

## Up
- [[Reconnaissance]]

Masscan is an asynchronous TCP port scanner built for **scale and speed** — it can scan the entire IPv4 internet in minutes by transmitting packets without waiting for replies (its own TCP/IP stack). Use it to find open ports fast across large ranges, then hand those ports to **[[Nmap]]** for deep service detection. Authorized scope only — its speed makes accidental out-of-scope scanning easy.

---

## Basic Usage

```bash
sudo masscan 10.0.0.0/24 -p80,443           # scan a /24 for two ports
sudo masscan 10.0.0.0/8 -p0-65535           # a /8 across all ports
sudo masscan 10.0.0.1 --top-ports 100
sudo masscan -iL targets.txt -p1-1000
sudo masscan 10.0.0.0/16 -p80,443 --exclude 10.0.5.0/24
```

Needs root (raw sockets). Ports: `-p80,443`, ranges `-p1-1000`, UDP with `-pU:53`, everything `-p0-65535`.

---

## Rate Control (the key knob)

```bash
sudo masscan 10.0.0.0/16 -p80 --rate 1000       # 1,000 packets/sec (gentle)
sudo masscan 10.0.0.0/8  -p80 --rate 100000     # 100k pps (fast, needs good NIC)
```

- `--rate` is packets per second — the single most important setting.
- Start low (1000) on networks you don't control; high rates can saturate links and trip IDS.
- Internet-scale scanning uses very high rates but requires a capable NIC and upstream.

---

## Output

```bash
sudo masscan 10.0.0.0/24 -p80,443 -oL out.list       # simple list
sudo masscan 10.0.0.0/24 -p80,443 -oJ out.json       # JSON
sudo masscan 10.0.0.0/24 -p80,443 -oX out.xml        # XML
sudo masscan 10.0.0.0/24 -p80,443 -oG out.grep       # grepable (nmap-like)

# resume an interrupted scan
sudo masscan 10.0.0.0/8 -p80 --rate 10000 -oL out.list   # writes paused.conf on Ctrl-C
sudo masscan --resume paused.conf
```

---

## Config Files & Banners

```bash
# save/reuse a config
sudo masscan 10.0.0.0/16 -p80,443 --rate 1000 --echo > scan.conf
sudo masscan -c scan.conf

# grab banners (basic service info) — needs extra care/permissions
sudo masscan 10.0.0.0/24 -p80 --banners
```

Masscan's banner grabbing is limited; for real version detection, use Nmap on the discovered ports.

---

## The Masscan → Nmap Handoff (standard pattern)

```bash
# 1. masscan finds open ports fast
sudo masscan 10.0.0.0/16 -p1-65535 --rate 10000 -oL mass.list

# 2. extract IP:port pairs, then deep-scan with Nmap
awk '/open/ {print $4}' mass.list | sort -u > live_ips.txt
ports=$(awk '/open/ {print $3}' mass.list | sort -un | paste -sd,)
sudo nmap -sV -sC -p "$ports" -iL live_ips.txt -oA nmap_deep
```

---

## Masscan vs Nmap

| | Masscan | Nmap |
|---|---|---|
| **Goal** | Find open ports across huge ranges, fast | Deep detail on specific hosts |
| **Speed** | Millions of pps possible | Much slower |
| **Service/version detection** | Minimal | Excellent (`-sV`, NSE) |
| **Best for** | Wide, shallow sweep | Narrow, deep enumeration |

Use them together: masscan for breadth, Nmap for depth.

---

## Tips & Cautions

- **Double-check scope and `--exclude`** before running — at high rates a typo can blast unintended networks in seconds.
- Syntax intentionally mirrors Nmap (`-p`, `-iL`, `-oX`) to ease switching.
- High rates can DoS fragile devices/links — ramp up gradually.
- It does host discovery + port scan in one async pass; there's no separate ping stage.
- Keep `paused.conf` — big scans get interrupted and `--resume` saves hours.
