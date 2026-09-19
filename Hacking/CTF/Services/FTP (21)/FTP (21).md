---
tags: [hacking, ctf, oscp, ftp]
type: cheatsheet
source: compiled reference (FTP enumeration)
last-verified: 2026-09-02
---

# FTP (21)

## Up
- [[Services]]

FTP transfers files, often over cleartext. CTF wins: **anonymous login**, readable/writable directories, credentials in files, version exploits, and cred reuse.

---

## Enumerate

```bash
nmap -p21 -sC -sV $IP                          # banner + anon-login NSE
nmap --script ftp-* -p21 $IP
nc -nv $IP 21                                   # manual banner grab
ftp $IP                                         # try: anonymous / anonymous (or blank)
```

Note the **software + version** (vsftpd, ProFTPD, FileZilla…) → `searchsploit`.

---

## Foothold Techniques

```bash
# anonymous access → download everything, recurse
ftp $IP    # user: anonymous, pass: anything
ftp> binary
ftp> ls -la
ftp> mget *
# or non-interactive:
wget -r ftp://anonymous:anon@$IP/

# writable dir + a web root? upload a webshell (if FTP dir == HTTP dir)
ftp> put shell.php     # then browse http://$IP/shell.php

# brute force (only if hinted — respect lockouts)
hydra -L users.txt -P rockyou.txt ftp://$IP -t 4
```

---

## Common Misconfigurations / Exploits

- **Anonymous login enabled** → read (creds, source, backups) and sometimes write.
- **Writable FTP root = web root** → upload a webshell → RCE.
- **Cleartext creds** sniffable; creds found here often **reused** on SSH/SMB.
- **vsftpd 2.3.4** backdoor (`:)` smiley → port 6200 shell) — `searchsploit vsftpd`.
- **ProFTPD 1.3.5** `mod_copy` (SITE CPFR/CPTO) → copy files to web root.
- Config/backups left in the share reveal usernames, DB creds, private keys.

---

## OSCP Tips

- Always try **anonymous** first; then look for a **writable** dir crossing into a web root.
- Grab **every file** — configs and backups here frequently unlock another port.
- Feed usernames/passwords found here into [[SSH (22)]], [[SMB (139,445)]], and [[Password Cracking]].
- Use **binary** mode for non-text files (keys, zips, exes).
