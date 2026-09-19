---
tags: [hacking, password-cracking, wordlists, seclists]
type: cheatsheet
source: compiled reference (wordlists / SecLists)
last-verified: 2026-08-30
---

# Wordlists

## Up
- [[Password Cracking]]

A password crack is only as good as its wordlist. This note covers the standard lists (**rockyou**, **SecLists**), where they live, and how to **generate custom, target-specific lists** — usually the highest-impact thing you can do beyond adding rules. Feeds [[Hashcat]], [[John the Ripper]], and [[Hydra]].

---

## The Classics

| List | What it is |
|---|---|
| **rockyou.txt** | ~14M real leaked passwords — the default go-to dictionary |
| **SecLists** | Huge curated collection: passwords, usernames, fuzzing, discovery, payloads |
| **CrackStation / HIBP** | Very large breach-derived lists (multi-GB) |
| **weakpass / hashesorg** | Aggregated breach + generated lists |

```bash
# common locations on Kali/pentest distros
/usr/share/wordlists/rockyou.txt          # may be rockyou.txt.gz → gunzip it
/usr/share/seclists/                       # apt install seclists  (or git clone danielmiessler/SecLists)
gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

## SecLists Structure (know these paths)

```text
SecLists/
├── Passwords/            # rockyou, breach lists, common creds, defaults
│   ├── Common-Credentials/
│   └── Leaked-Databases/
├── Usernames/            # name lists, service accounts
├── Discovery/            # web content & DNS enumeration
│   ├── Web-Content/  (directory-list-2.3-medium.txt, raft-*)
│   └── DNS/          (subdomains-top1million-*.txt)
├── Fuzzing/              # payloads for fuzzers
└── Web-Shells/  Miscellaneous/  Pattern-Matching/
```

- Passwords → [[Hashcat]]/[[John the Ripper]]/[[Hydra]]
- Usernames → spraying/Hydra `-L`
- Discovery/DNS → [[OSINT]] subdomain brute-force ([[Amass]])
- Discovery/Web-Content → gobuster/ffuf/dirb

---

## Generating Custom Wordlists

### crunch — generate by pattern

```bash
crunch 8 8 -t @@@@%%%% -o out.txt    # 8 chars: 4 lower + 4 digits
crunch 6 8 abcdefg123 -o out.txt     # min6 max8 from a charset
# placeholders: @ lower, , upper, % digit, ^ symbol
```

### cewl — scrape a target's website for words

```bash
cewl -d 2 -m 5 -w site.txt https://example.com     # depth 2, min length 5
cewl --with-numbers -w site.txt https://example.com
```

Company/product/jargon words from the target site make excellent base words (people use what's around them).

### CUPP / username-based

```bash
cupp -i          # interactive: builds a list from a person's name, DOB, pet, etc. (OSINT)
```

### Mangle & mutate with rules
Rather than storing giant lists, keep a solid base list and let the cracker expand it:

```bash
hashcat ... rockyou.txt -r rules/best64.rule       # rules multiply the base list
john --wordlist=base.txt --rules --stdout > mutated.txt
```

---

## Curating & Combining

```bash
sort -u a.txt b.txt > merged.txt            # merge + dedupe
awk 'length>=8 && length<=16' rockyou.txt > filtered.txt   # length filter to policy
grep -iE '2024|2025|2026|!' rockyou.txt > seasonal.txt
# hashcat can filter to a password policy while cracking too
```

- Trim to the target's **password policy** (min length, required classes) to cut wasted guesses.
- Add **seasonal/local** terms (Company2026!, city/sports-team names) — humans are predictable.

---

## Strategy — pick the right list for the job

| Job | Start with |
|---|---|
| Generic offline hash | rockyou + rules |
| Known policy | length-filtered list + mask ([[Password Attack Types]]) |
| Target-specific | cewl (site) + CUPP (people) + rockyou, all + rules |
| Online spraying | tiny list of 3-5 common/seasonal passwords, many users |
| Web/dir discovery | SecLists Discovery/Web-Content |
| Subdomain brute | SecLists Discovery/DNS |

---

## Tips

- **rockyou + a good rule file** (best64 / OneRuleToRuleThemAll) beats a bigger raw list most of the time.
- Custom > generic: a small list built from the target (cewl + CUPP) often cracks what rockyou can't.
- Don't store terabytes — keep base lists and generate variants with **rules** at crack time.
- Match list size to hash speed: for slow hashes (bcrypt/argon2) a small, smart list is essential; for fast NTLM/MD5 you can afford big lists + heavy rules.
