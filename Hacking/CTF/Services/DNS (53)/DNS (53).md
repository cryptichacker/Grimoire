---
tags: [hacking, ctf, oscp, dns]
type: cheatsheet
source: compiled reference (DNS enumeration)
last-verified: 2026-09-02
---

# DNS (53)

## Up
- [[Services]]

DNS on a target (especially a **Domain Controller**) leaks hostnames and internal structure. CTF wins: **zone transfers**, subdomain/vhost discovery, and confirming a Windows AD domain.

---

## Enumerate

```bash
nmap -p53 -sC -sV $IP
dig @$IP version.bind chaos txt         # sometimes leaks the DNS software
dig @$IP any corp.local                 # records

# ZONE TRANSFER (AXFR) — the jackpot when allowed
dig axfr @$IP corp.local
dnsrecon -d corp.local -n $IP -t axfr

# reverse lookup
dig -x $IP @$IP

# subdomain brute (if you have a domain name)
dnsrecon -d corp.local -D subdomains.txt -t brt -n $IP
gobuster dns -d corp.local -r $IP -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## Foothold Techniques

DNS is usually **not** a direct foothold — it feeds others:

- **Zone transfer** → all hostnames/subdomains → add to `/etc/hosts` → new web vhosts, new services.
- Confirms **AD**: DC often runs DNS (53) alongside 88/389/445 → go [[Active Directory]].
- Discovered vhosts → re-scan/fuzz each ([[HTTP (80,443)]]).

---

## Common Misconfigurations

- **AXFR allowed to anyone** → full zone dump (huge info leak).
- Internal hostnames revealing infrastructure (dev/staging/admin subdomains).
- Wildcard records / info that maps the environment.

---

## OSCP Tips

- Always try **AXFR** (`dig axfr @$IP <domain>`) — quick and occasionally hands you everything.
- Add every discovered hostname to **`/etc/hosts`**, then re-run web enumeration against the names (vhost routing).
- DNS + Kerberos(88) + LDAP(389) = **Domain Controller** → pivot to [[Active Directory]].
