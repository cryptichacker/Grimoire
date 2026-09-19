---
tags: [hacking, ctf, oscp, snmp]
type: cheatsheet
source: compiled reference (SNMP enumeration)
last-verified: 2026-09-02
---

# SNMP (161)

## Up
- [[Services]]

SNMP (UDP 161) is a goldmine that's easy to miss (UDP!). With a valid **community string** (often `public`), it leaks users, running processes (sometimes **with command-line creds**), installed software, network info, and open ports.

---

## Enumerate

```bash
# find it (UDP!)
sudo nmap -sU -p161 -sV $IP

# guess the community string
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP

# walk the MIB tree with a valid string (usually 'public')
snmpwalk -v2c -c public $IP                     # everything (noisy but rich)
snmpwalk -v2c -c public $IP 1.3.6.1.4.1.77.1.2.25    # user accounts
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.25.4.2.1.2   # running processes
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.25.4.2.1.5   # process command lines (CREDS!)
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.25.6.3.1.2   # installed software
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.6.13.1.3     # listening TCP ports

snmp-check $IP -c public                         # nicely formatted summary
```

---

## Foothold Techniques

- **Process command lines** often contain passwords (`mysql -u root -pSecret`, backup scripts) → reuse on [[SSH (22)]]/[[MySQL (3306)]]/etc.
- **Usernames** → user list for spraying / [[Active Directory]].
- **Installed software + versions** → `searchsploit`.
- **Internal listening ports** → services to pivot to.
- **SNMP RW community** (rarer) → change config (e.g. on network gear) for code exec.

---

## Common Misconfigurations

- Default community strings **`public`** (RO) / **`private`** (RW) left enabled.
- SNMP **v1/v2c** (no encryption/auth beyond the string) exposing everything.
- Extend MIBs (`nsExtendObjects`) allowing command output exposure.

---

## OSCP Tips

- **Don't forget UDP** — SNMP is invisible to a TCP-only scan and often holds the key.
- Focus the walk on **process args** (`...25.4.2.1.5`) — that's where creds hide.
- Try `public`, `private`, and community strings hinted by the box name/theme.
- Everything found here (users, creds, ports) feeds other services — it's an enumeration multiplier.
