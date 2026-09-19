---
tags: [hacking, password-cracking, hydra, online]
type: cheatsheet
source: compiled reference (THC-Hydra)
last-verified: 2026-08-30
---

# Hydra

## Up
- [[Password Cracking]]

THC-Hydra is a fast **online** (network) login brute-forcer. Unlike [[Hashcat]]/[[John the Ripper]] (which crack hashes offline), Hydra throws username/password guesses at **live services** — SSH, FTP, RDP, SMB, HTTP forms, databases, and many more. It's inherently noisy: authorized scope only, and mind lockout policies.

---

## Command Shape

```bash
hydra [-l user | -L users.txt] [-p pass | -P passwords.txt] [options] <target> <service>
```

- `-l` single user / `-L` user list · `-p` single password / `-P` password list.
- `-C combos.txt` uses `user:pass` lines instead of separate `-L`/`-P`.

---

## Common Options

| Flag | Meaning |
|---|---|
| `-l` / `-L` | Single login / login list |
| `-p` / `-P` | Single password / password list |
| `-C file` | Colon-separated `user:pass` combo list |
| `-t N` | Parallel tasks (default 16; lower to be gentle) |
| `-f` | Stop after the first valid pair (per host) |
| `-s PORT` | Non-default port |
| `-V` / `-vV` | Verbose (show each attempt) |
| `-o file` | Write found credentials to a file |
| `-e nsr` | Also try **n**ull, **s**=login-as-password, **r**=reversed |
| `-u` | Loop users outer (better vs lockouts) |
| `-w` | Wait/timeout tuning |

---

## Service Examples

```bash
# SSH
hydra -l root -P rockyou.txt ssh://10.0.0.1
hydra -L users.txt -P pass.txt -t 4 ssh://10.0.0.1

# FTP
hydra -l admin -P pass.txt ftp://10.0.0.1

# RDP (go slow — locks accounts fast)
hydra -l admin -P pass.txt rdp://10.0.0.1 -t 1

# SMB
hydra -L users.txt -P pass.txt smb://10.0.0.1

# database
hydra -l sa -P pass.txt mysql://10.0.0.1
hydra -l postgres -P pass.txt postgres://10.0.0.1
```

---

## HTTP Login Forms (the tricky one)

### POST form — `http-post-form`

```bash
hydra -l admin -P rockyou.txt 10.0.0.1 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid credentials"
```

The module string has **three colon-separated parts**:
1. **path** — `/login`
2. **POST body** with `^USER^` / `^PASS^` placeholders
3. **failure condition** — `F=<text on failed login>` (or `S=<text on success>`)

```bash
# GET form
hydra ... http-get-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# HTTPS form (note https-post-form)
hydra ... https-post-form "/login:u=^USER^&p=^PASS^:Invalid"

# with a required cookie / CSRF header
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=fail:H=Cookie: sess=abc123"

# HTTP Basic auth
hydra -l admin -P pass.txt 10.0.0.1 http-get /admin
```

Get the exact field names, path, and the failure/success string from Burp/DevTools first — the `F=`/`S=` marker is what tells Hydra whether a guess worked.

---

## Staying Out of Trouble (lockouts & noise)

- **Lockout policy** — many systems lock after 3–5 failures. Brute-forcing one account can lock it (a DoS). Prefer **password spraying**: one/few passwords across many users, with `-u` and low `-t`, spaced out.
- **Throttle**: `-t 1`–`4` and add waits; RDP/SMB especially.
- **MFA** blocks Hydra entirely — a valid password still won't log in.
- Everything is logged on the target — expect detection.

Alternatives with similar roles: **medusa**, **ncrack**, **patator**, **crackmapexec** (SMB/WinRM spraying).

---

## Tips

- Confirm the service, port, and (for web) the exact request with [[Reconnaissance]]/Burp before launching.
- Build username lists from [[OSINT]] (theHarvester) and passwords from [[Wordlists]].
- Use `-e nsr` to cheaply catch blank passwords and username-equals-password.
- `-f` stops on first hit per host; `-o found.txt` saves results.
- For web forms, get the `F=`/`S=` marker right — a wrong marker makes Hydra report all or no successes.
