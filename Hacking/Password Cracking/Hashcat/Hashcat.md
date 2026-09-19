---
tags: [hacking, password-cracking, hashcat]
type: cheatsheet
source: compiled reference (hashcat)
last-verified: 2026-08-30
---

# Hashcat

## Up
- [[Password Cracking]]

Hashcat is the fastest **offline** password recovery tool — GPU-accelerated, supporting hundreds of hash algorithms and every major attack mode (dictionary, rules, mask, hybrid, brute-force). Crack only hashes you're authorized to hold.

---

## Command Shape

```bash
hashcat -m <hash-mode> -a <attack-mode> [options] <hashfile> [wordlist|mask]
```

- `-m` = hash type (numeric mode); `-a` = attack mode.
- Hashes usually go in a **file** (one per line); with usernames use `--username`.

---

## Attack Modes (`-a`)

| Mode | Attack | Extra input |
|---|---|---|
| `0` | Dictionary (wordlist) | a wordlist |
| `1` | Combinator (word1+word2) | two wordlists |
| `3` | Mask (brute-force by pattern) | a mask |
| `6` | Hybrid: wordlist + mask | wordlist + mask |
| `7` | Hybrid: mask + wordlist | mask + wordlist |
| `9` | Association | — |

## Hash Modes (`-m`, common)

| Mode | Algorithm |
|---|---|
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA-256 |
| `1000` | NTLM (Windows) |
| `3000` | LM |
| `5600` | NetNTLMv2 (Responder captures) |
| `1800` | sha512crypt (Linux `$6$`) |
| `3200` | bcrypt (`$2*$`) — slow |
| `13100` | Kerberos TGS-REP (Kerberoast) |
| `18200` | Kerberos AS-REP (AS-REP roast) |
| `2500`/`22000` | WPA/WPA2 (PMKID/handshake) |
| `22100` | BitLocker |

```bash
hashcat --identify hashes.txt      # suggest the mode
hashcat -h | grep -i ntlm          # search modes
```

---

## Core Examples

```bash
# dictionary
hashcat -m 0 -a 0 hashes.txt rockyou.txt

# dictionary + rules (biggest bang for buck)
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt -r rules/best64.rule

# mask / brute-force (8 chars: Upper + lower*5 + 2 digits)
hashcat -m 0 -a 3 hashes.txt '?u?l?l?l?l?l?d?d'

# pure brute-force, all chars, lengths 1-8
hashcat -m 0 -a 3 hashes.txt '?a?a?a?a?a?a?a?a' --increment

# hybrid: word + 4-digit suffix (e.g. summer2024)
hashcat -m 0 -a 6 hashes.txt rockyou.txt '?d?d?d?d'
```

### Mask charsets

| Token | Set |
|---|---|
| `?l` | a-z |
| `?u` | A-Z |
| `?d` | 0-9 |
| `?s` | symbols |
| `?a` | all of the above |
| `?b` | 0x00-0xff |

Custom sets: `-1 ?l?d` defines `?1`; then use `?1?1?1?1`.

---

## Rules (mangling)

```bash
hashcat -m 0 -a 0 hashes.txt words.txt -r rules/best64.rule
hashcat ... -r rules/best64.rule -r rules/toggles1.rule   # stack multiple
hashcat ... --stdout -r rules/best64.rule < words.txt      # preview mangled words
```

Bundled favourites: `best64.rule`, `rockyou-30000.rule`, `dive.rule`, `OneRuleToRuleThemAll.rule`.

---

## Running, Sessions & Results

```bash
hashcat -m 1000 -a 0 h.txt rockyou.txt -O -w 3            # -O optimized kernel, -w workload 1-4
hashcat ... --session=job1                                 # named session
hashcat ... --restore --session=job1                       # resume after interrupt
hashcat -m 1000 h.txt --show                               # show already-cracked
hashcat -m 1000 h.txt --left                               # show still-uncracked
hashcat ... --potfile-path job.pot                         # cracked pairs stored in potfile
hashcat -b                                                 # benchmark your GPU
hashcat -m 1000 h.txt rockyou.txt -o cracked.txt           # write results to file
```

Interactive keys while running: `s` status, `p` pause, `r` resume, `q` quit, `b` bypass current attack.

---

## Capturing Hashes for `-m` modes (examples)

- **NetNTLMv2** → Responder/Impacket capture → `-m 5600`.
- **Kerberoast** → `GetUserSPNs.py` (Impacket) → `-m 13100`.
- **AS-REP roast** → `GetNPUsers.py` → `-m 18200`.
- **WPA2** → `hcxdumptool`/`hcxpcapngtool` → `.22000` → `-m 22000`.
- **Linux shadow** → `unshadow` (from John) → detect `$6$` = `-m 1800`.

---

## Tips

- **Order of escalation:** wordlist → wordlist+rules → hybrid → mask → brute-force.
- `-O` (optimized) is much faster but caps password length (~31) — fine for most cases.
- GPUs crush **fast** hashes (MD5/NTLM/SHA); for **bcrypt/argon2** rely on smart wordlists + rules, not brute-force.
- Keep the **potfile** — hashcat auto-skips already-cracked hashes across runs.
- Build target-specific wordlists from [[OSINT]] and combine with [[Wordlists]] + rules for the best hit rate.
- John and Hashcat complement each other — see [[John the Ripper]] for its `*2john` extractors that produce hashes hashcat can eat.
