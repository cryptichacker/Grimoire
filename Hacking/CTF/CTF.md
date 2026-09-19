---
tags: [hacking, ctf, oscp, methodology]
type: moc
---

# CTF

## Up
- [[Hacking]]

A practical **CTF / boot-to-root methodology** for TryHackMe, HackTheBox, and **OSCP** lab practice — the repeatable flow from a fresh IP to `root.txt`. Open the visual **`CTF Methodology`** canvas for the map; use the notes below for the detail at each step.

> **Scope:** for **authorized** lab/training targets only (THM/HTB/PWK labs, your own VMs, the OSCP exam). This is an educational methodology reference.

## The Flow

```mermaid
flowchart LR
    A[Recon &<br/>Enumeration] --> B{Open ports?}
    B --> S[Per-service<br/>enumeration]
    S --> C[Initial Foothold<br/>reverse shell]
    C --> D[Stabilize Shell<br/>full TTY]
    D --> E[Local Enumeration]
    E --> F[Privilege<br/>Escalation]
    F --> G[root / SYSTEM<br/>+ loot]
    F -. lateral / AD .-> H[Domain Admin]
    S -. user.txt .-> Flag[(Flags)]
    F -. root.txt .-> Flag
```

## Phases
- [[Recon & Enumeration]] — nmap, port triage, "what do I do with each open port?"
- [[Services]] — per-port enumeration, foothold techniques & common misconfigs (the heart of this)
- [[Shell Stabilization]] — upgrade a dumb reverse shell to a full interactive TTY
- [[Privilege Escalation]] — local enum → root/SYSTEM ([[Linux PrivEsc]] · [[Windows PrivEsc]] · [[Active Directory]])

## Related (from the rest of Hacking)
- [[Reconnaissance]] ([[Nmap]]) · [[OSINT]] · [[Password Cracking]] ([[Hashcat]], [[John the Ripper]], [[Hydra]], [[Wordlists]])
- [[Web]] (PortSwigger/OWASP) — deep web-exploitation theory behind the HTTP node
- [[Methodology]] — PTES/ATT&CK framing (CTFs are a compressed PTES engagement)

---

## The OSCP Mindset (try-harder, but methodically)

1. **Enumerate more than you think you need to.** Most "stuck" moments are missed enumeration, not missing skills.
2. **Every open port is a door** — for each, ask: *version → known exploits? → default/weak creds? → misconfig? → info leak that helps another port?*
3. **Keep notes per target** (IP, ports, versions, creds found, foothold, privesc). Reuse creds **everywhere** (password reuse is the #1 CTF theme).
4. **Foothold → stabilize → enumerate again as the new user → escalate.** Re-enumerate after every privilege change.
5. **Manual first, automated second.** Know what LinPEAS/WinPEAS *find* so you can find it by hand when tools fail.
6. **Exam note (current OSCP):** heavy on **Active Directory** + standalone Linux/Windows; classic 32-bit stack **buffer overflow is no longer in the exam**. Time-box each target and rotate.

---

## Quick "Open Port → Go To" Index

| Port(s) | Service | Note |
|---|---|---|
| 21 | FTP | [[FTP (21)]] |
| 22 | SSH | [[SSH (22)]] |
| 25/110/143 | SMTP/POP3/IMAP | [[SMTP & Mail (25,110,143)]] |
| 53 | DNS | [[DNS (53)]] |
| 80/443 | HTTP/HTTPS | [[HTTP (80,443)]] |
| 111/135/139/445 | RPC/NetBIOS/SMB | [[SMB (139,445)]] |
| 161 | SNMP | [[SNMP (161)]] |
| 389/636 | LDAP | [[LDAP (389)]] |
| 1433 | MSSQL | [[MSSQL (1433)]] |
| 2049 | NFS | [[NFS (2049)]] |
| 3306 | MySQL | [[MySQL (3306)]] |
| 3389 | RDP | [[RDP (3389)]] |
| 5432 | PostgreSQL | [[PostgreSQL (5432)]] |
| 5985/5986 | WinRM | [[WinRM (5985,5986)]] |
| 6379 | Redis | [[Redis (6379)]] |
| *others* | Telnet/VNC/rsync/Kerberos/Elastic/Mongo… | see the table in [[Services]] |
