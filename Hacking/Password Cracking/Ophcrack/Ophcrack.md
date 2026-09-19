---
tags: [hacking, password-cracking, ophcrack, rainbow-tables, windows]
type: cheatsheet
source: compiled reference (Ophcrack)
last-verified: 2026-08-30
---

# Ophcrack

## Up
- [[Password Cracking]]

Ophcrack is a free **rainbow-table** password cracker specialised for **Windows LM and NTLM** hashes. Instead of computing hashes on the fly, it looks plaintexts up in massive **precomputed tables** — a time-memory trade-off that recovers many Windows passwords in seconds to minutes. It even ships as a bootable live CD/USB that dumps and cracks local SAM hashes. Authorized systems only. See [[Password Attack Types]] for the rainbow-table concept.

---

## How Rainbow Tables Work (recap)

- Precompute chains of hash→reduce→hash across a keyspace and store only chain endpoints → a compact lookup structure.
- Cracking = regenerate a chain from the target hash and match an endpoint, then walk the chain to the plaintext.
- **Time-memory trade-off:** huge disk (GBs–TBs of tables) in exchange for near-instant lookups vs. live brute-force.
- **Only works on UNSALTED hashes.** Windows LM/NTLM are unsalted → perfect targets. Modern salted hashes (bcrypt/argon2, Linux `$6$`) are **immune** — use [[Hashcat]]/[[John the Ripper]] instead.

### Why Windows LM is so weak
Legacy **LM** hashes uppercase the password, split it into two 7-char halves, and hash each separately — so any password ≤14 chars is really two ≤7-char uppercase problems. Rainbow tables shred it. **NTLM** is stronger (case-sensitive, no split) but still **unsalted**, so tables work up to a length/charset the table covers. Disable LM everywhere.

---

## Using Ophcrack

### GUI workflow
1. Install tables (Tables menu → Install → point at a downloaded set).
2. Load hashes: **Load** →
   - *Single hash* / *PWDUMP file* (from pwdump/fgdump/samdump2), or
   - *Encrypted SAM* (a copy of `SAM` + `SYSTEM` hives), or
   - *Local SAM* (live system, admin).
3. Click **Crack**. Recovered passwords appear in the NT/LM Pwd columns.

### Bootable Live CD/USB
Boot the target from Ophcrack Live → it auto-locates the Windows SAM, extracts hashes, and cracks them with the bundled free tables — no login needed (physical-access scenario).

---

## Rainbow Table Sets

| Table set | Covers | Cost |
|---|---|---|
| **XP free small / fast** | LM, alphanumeric | Free |
| **Vista/7/8/10 free** | NTLM, common passwords | Free |
| **Larger "special"/"pro" tables** | Bigger charset/length (symbols, longer) | Paid |

Bigger tables = more coverage (more length/charset) but far more disk and download. LM tables are small and devastating; NTLM tables get large quickly.

---

## Getting Windows Hashes to Feed It

```text
# from a live/offline Windows system (authorized):
- samdump2 SYSTEM SAM > hashes.txt        # from copied SAM + SYSTEM hives
- pwdump / fgdump                         # classic dumpers → PWDUMP format
- Impacket secretsdump.py                 # secretsdump -> NTLM hashes
- reg save HKLM\SAM sam.hiv & reg save HKLM\SYSTEM sys.hiv  # copy hives (admin)
```

Ophcrack loads PWDUMP-format files or encrypted SAM (SAM+SYSTEM) directly.

---

## Ophcrack vs Pure Crackers

| | Ophcrack (rainbow tables) | [[Hashcat]] / [[John the Ripper]] |
|---|---|---|
| Method | Precomputed lookup | Compute-on-the-fly |
| Works on salted hashes? | **No** | Yes |
| Speed on unsalted LM/NTLM | Near-instant (if in table) | Fast, but recomputes |
| Coverage limit | What the table includes | Wordlist/rules/mask you choose |
| Disk cost | Very high (tables) | Low |

Use Ophcrack when you have **unsalted Windows hashes** and the password is likely within a table's charset/length; fall back to Hashcat/John (wordlist + rules + mask) for anything salted, longer, or outside the table.

---

## Tips & Defensive Notes

- **Salting kills rainbow tables** — the whole reason modern systems salt hashes. This is why Ophcrack is Windows-LM/NTLM-specific.
- **Blue team:** disable LM hash storage (`NoLMHash` policy), enforce long passphrases (beyond table coverage), and never store unsalted hashes.
- Rainbow tables only cover the charset/length they were built for — a 16-char NTLM password with symbols usually isn't in any practical table.
- For salted or long hashes, don't waste time here — go straight to [[Hashcat]] with good [[Wordlists]] + rules.
