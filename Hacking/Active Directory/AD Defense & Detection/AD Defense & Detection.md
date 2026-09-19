---
tags: [hacking, active-directory, windows, defense, detection, blue-team]
type: cheatsheet
source: personal Notion — "Active Directory" cheatsheet + additions
last-verified: 2026-09-03
---

# AD Defense & Detection

## Up
- [[Active Directory]]

The blue-team side — the Event IDs, hardening, and controls that detect or prevent the attacks in this category. Useful for reporting (map each finding to a detection/mitigation) and for hardening your own labs.

---

## Critical Event IDs

**Authentication**
```text
4768  Kerberos TGT request (AS-REQ)
      ├─ preauth type 0        → AS-REP roasting
      ├─ result 0x6            → username enumeration (bad user)
      └─ result 0x18           → bad password (spray)
4769  Kerberos service ticket (TGS-REQ)
      └─ RC4 (etype 0x17) + many SPNs from one host → Kerberoasting
4771  Kerberos pre-auth failed → brute/spray (many accounts, one source)
4776  DC validated NTLM creds → NTLM where Kerberos expected (PtH/relay)
4672  Special privileges assigned at logon → admin/service usage
4624 / 4625  Logon success / failure (type 3 = network)
```
**Account & object changes**
```text
4720/4722/4725  user created/enabled/disabled
4724            password reset
4728/4732/4756  member added to (global/local/universal) security group
4740            account locked out
4662            operation on object → DCSync (repl GUID 1131f6aa-…), AdminSDHolder
5136            directory object modified → ACL/attribute/trust changes
4886/4887       AD CS certificate requested/issued → ESC abuse
```

### Attack → signal quick map
| Attack | Look for |
|---|---|
| AS-REP roast | 4768 preauth type 0 |
| Kerberoast | 4769 RC4 + many SPNs |
| Password spray | many 4771/4625 across accounts, one source |
| Pass-the-Hash | 4776 / 4624 type 3 with NTLM |
| Pass-the-Ticket | 4624 with no preceding 4768 |
| DCSync | 4662 replication rights from non-DC |
| Golden Ticket | 4624 no 4768; odd lifetimes; non-existent users |
| ACL abuse | 5136 on sensitive objects/ACLs |
| ADCS ESC | 4886/4887 unusual SAN/template |

---

## Hardening

**Accounts**
- Strong policy: ≥14-char (passphrases), history 24, max age 60–90d, lockout 5 / 30 min.
- **Service accounts → gMSA** (120-char auto password, 30-day rotation) or ≥25-char passwords; dedicated per service; no interactive logon → kills Kerberoast/Silver value.
- **Tiered admin model** (Tier 0/1/2): separate admin accounts, no DA on workstations, **PAWs** for Tier 0, no internet/email on admin accounts.

**Kerberos / NTLM**
- Enforce **AES** (disable RC4) → weakens roasting/overpass; enable PAC validation → weakens silver tickets.
- **Disable NTLM** where possible; require **SMB signing** and **LDAP signing + channel binding** → kills relay ([[NTLM Relay & Coercion]]).
- Disable **LLMNR/NBT-NS/mDNS**; block rogue DHCPv6/IPv6.

**Delegation / ACLs / ADCS**
- Avoid **unconstrained delegation**; audit `msDS-AllowedToActOnBehalfOf…`; set `MachineAccountQuota = 0` (blocks easy RBCD).
- Review dangerous ACLs (WriteDACL/GenericAll) on admins/domain; watch `msDS-KeyCredentialLink` (shadow creds).
- Audit AD CS templates (no enrollee SAN on client-auth templates, manager approval), disable HTTP + enable **EPA** on Web Enrollment (ESC8).

**Persistence hygiene**
- **Rotate krbtgt twice** periodically and after any DA compromise (invalidates golden tickets).
- **LAPS** for unique local-admin passwords (defeats PtH lateral movement).
- Monitor AdminSDHolder ACL, `sIDHistory`, DSRM behavior, rogue replication (DCShadow), and issued certs.

---

## Tooling (audit your own)
- **BloodHound** (find the paths before attackers do), **PingCastle** / **Purple Knight** (posture scoring), **ADeleg** (delegation audit), **Certipy find** (ADCS), Microsoft **LAPS**/gMSA, and a SIEM watching the Event IDs above.

---

## Takeaways
- Most attacks have a clear **Event ID signature** — 4768/4769/4771/4776/4662 cover roasting, spraying, PtH, and DCSync.
- **gMSA + AES + SMB/LDAP signing + no unconstrained delegation + MAQ 0 + LAPS + tiering** removes the bulk of the paths in this category.
- After any DA compromise: **rotate krbtgt twice**, reset trust keys, and hunt the persistence in [[Trusts & Persistence]].
