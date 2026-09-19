---
tags: [hacking, ctf, oscp, ssh]
type: cheatsheet
source: compiled reference (SSH enumeration)
last-verified: 2026-09-02
---

# SSH (22)

## Up
- [[Services]]

SSH is rarely the *initial* way in on its own — it's usually how you **use credentials/keys found elsewhere**. Wins: cred reuse, exposed private keys, weak passwords, and version-specific bugs.

---

## Enumerate

```bash
nmap -p22 -sC -sV $IP                 # banner (OpenSSH version), host keys, auth methods
nc -nv $IP 22                          # manual banner
ssh -v user@$IP                        # see offered auth methods (publickey/password)
```

Version → `searchsploit openssh`. Note supported auth (key vs password).

---

## Foothold Techniques

```bash
# creds found elsewhere (FTP/HTTP/DB/SMB) — TRY THEM
ssh user@$IP

# private key looted from web/FTP/LFI:
chmod 600 id_rsa
ssh -i id_rsa user@$IP
# key has a passphrase? crack it:
ssh2john id_rsa > id.hash && john --wordlist=rockyou.txt id.hash

# password brute (only when hinted; watch lockout)
hydra -L users.txt -P rockyou.txt ssh://$IP -t 4
```

---

## Common Misconfigurations / Exploits

- **Weak/default passwords**, or password auth left on for a user you found.
- **Exposed `id_rsa`** via LFI/web/backup → direct login (crack passphrase if set).
- **`authorized_keys` writable** (via another vuln) → drop your key.
- **User enumeration** on old OpenSSH (`CVE-2018-15473`) → valid usernames.
- **Reused credentials** — the #1 SSH path in CTFs.
- **Restricted shells** (rbash) — escape via `vi`, `python`, or `ssh user@host -t "bash --noprofile"`.

---

## OSCP Tips

- Think of SSH as an **exit** for creds/keys you collected, not a place to brute blindly.
- Looted keys: `chmod 600` and try each discovered **username** with the key.
- In an rbash/restricted shell, look for GTFOBins-style escapes.
- After login → [[Shell Stabilization]] is already stable (real TTY) → go to [[Linux PrivEsc]].
