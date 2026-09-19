---
tags: [hacking, password-cracking, john]
type: cheatsheet
source: compiled reference (John the Ripper / Jumbo)
last-verified: 2026-08-30
---

# John the Ripper

## Up
- [[Password Cracking]]

John the Ripper ("John", JtR) is a veteran **offline** password cracker. The community **"jumbo"** build adds hundreds of hash formats and a huge suite of `*2john` tools that extract crackable hashes from files (ZIP, PDF, SSH keys, KeePass, etc.). Great autodetection and CPU strength; pairs with [[Hashcat]] for GPU. Authorized hashes only.

---

## Basic Usage

```bash
john hashes.txt                       # autodetect format + default attacks
john --wordlist=rockyou.txt hashes.txt
john --format=raw-md5 hashes.txt      # force a format
john --show hashes.txt                # display cracked passwords
john --show --format=nt hashes.txt
```

John auto-picks a format and runs single → wordlist → incremental if you don't specify. Results are stored in `~/.john/john.pot`.

---

## Cracking Modes

```bash
# single crack (uses usernames/GECOS as hints — very effective first pass)
john --single hashes.txt

# wordlist
john --wordlist=rockyou.txt hashes.txt

# wordlist + rules (mangling)
john --wordlist=rockyou.txt --rules hashes.txt
john --wordlist=rockyou.txt --rules=Jumbo hashes.txt

# incremental (smart brute-force by frequency — can run "forever")
john --incremental hashes.txt
john --incremental=Digits hashes.txt

# mask (targeted brute-force, hashcat-style placeholders)
john --mask='?u?l?l?l?l?d?d' hashes.txt
```

| Mode | Flag | Notes |
|---|---|---|
| Single | `--single` | Username-derived guesses; always run first |
| Wordlist | `--wordlist=` | Dictionary; add `--rules` |
| Incremental | `--incremental` | Statistical brute-force |
| Mask | `--mask=` | Pattern brute-force |
| External | `--external=` | Custom filters in john.conf |

---

## Format Detection & Listing

```bash
john --list=formats | tr ',' '\n' | grep -i sha
john --list=format-details --format=nt
# hashid / hash-identifier also help identify before choosing --format
```

---

## The `*2john` Extractors (John's superpower)

Convert protected files into hashes John/Hashcat can crack:

```bash
zip2john secret.zip    > zip.hash
rar2john archive.rar   > rar.hash
pdf2john document.pdf  > pdf.hash
ssh2john id_rsa        > ssh.hash        # passphrase-protected SSH key
keepass2john db.kdbx   > kp.hash
office2john doc.docx   > office.hash
7z2john file.7z        > 7z.hash

john --wordlist=rockyou.txt zip.hash
```

Also: `unshadow` merges `/etc/passwd` + `/etc/shadow` for Linux cracking:

```bash
unshadow passwd.txt shadow.txt > unshadowed.txt
john --wordlist=rockyou.txt unshadowed.txt
```

Dozens more exist (`bitlocker2john`, `gpg2john`, `mozilla2john`, …) — check `ls /usr/share/john/ | grep 2john` or `*2john`.

---

## Sessions & Control

```bash
john --session=job1 --wordlist=rockyou.txt hashes.txt
john --restore=job1                    # resume
john --status=job1
# press any key during a run to print status
john --pot=custom.pot hashes.txt
```

---

## Rules

Rules live in `john.conf` (`[List.Rules:...]`). Common named sets: `Single`, `Wordlist`, `Jumbo`, `KoreLogic`, `All`.

```bash
john --wordlist=words.txt --rules=KoreLogic hashes.txt
john --wordlist=words.txt --rules --stdout          # preview mangled candidates
```

---

## John vs Hashcat

| | John the Ripper | [[Hashcat]] |
|---|---|---|
| Engine | CPU-focused (jumbo has some GPU) | GPU-first, fastest |
| Autodetection | Excellent (`john hashes.txt` just works) | Manual `-m` mode |
| File extractors | Huge `*2john` suite | Relies on external tools |
| Best for | Quick start, odd formats, single/rules | Raw speed on big/fast-hash jobs |

Common combo: use `*2john` to extract, then crack with **Hashcat** on GPU for speed.

---

## Tips

- Always run `--single` first — username/GECOS-based guesses are cheap and hit often.
- `--show` reads the potfile; John skips already-cracked hashes automatically.
- Point `--wordlist` at good lists (see [[Wordlists]]) and layer `--rules`.
- For GPU speed on large jobs, extract with John's tools and finish in [[Hashcat]].
