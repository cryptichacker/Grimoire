---
tags: [hacking, active-directory, windows, theory]
type: cheatsheet
source: personal Notion — "Active Directory" cheatsheet
last-verified: 2026-09-03
---

# AD Fundamentals

## Up
- [[Active Directory]]

The theory you must know before attacking AD — structure, components, objects, trusts, and the NTLM/Kerberos authentication internals that every attack abuses.

---

## What is Active Directory?

Microsoft's directory service on Windows Server — a centralized database + services connecting users to network resources (a "phone book" for the network storing users, computers, printers, etc.). First shipped with Windows 2000, replacing the flat NT domain model with a hierarchical, object-based design.

**Core purposes:** Authentication (who are you?), Authorization (what can you access?), centralized management, policy enforcement (GPO), service location.
**Benefits:** Single Sign-On, centralized admin, scalability, granular security, fault tolerance (multiple DCs).

### Domain structure

```text
Forest: contoso.com (Complete AD Instance)
├── Root Domain: contoso.com
│   ├── Domain Controller (DC1/DC2.contoso.com)
│   ├── OU: IT Department → Users / Computers / Groups
│   ├── OU: Sales Department
│   └── Built-in: Domain Admins, Enterprise Admins, Administrator
├── Child Domain: dev.contoso.com   (shares namespace, auto two-way transitive trust)
└── Tree: subsidiary.com            (separate namespace, same forest)
```

- **Forest** — top container; shares schema, configuration, global catalog; automatic two-way transitive trusts between its domains. **The forest is the true security boundary.**
- **Domain** — its own auth database, policies, admin control.
- **OU** — organizes objects; unit of GPO application and admin delegation.
- **Namespace (DNS-based):** UPN `alice@contoso.com`; Distinguished Name `CN=alice,OU=IT,DC=contoso,DC=com`.

---

## Components

### Domain Controllers (DC)
Windows Servers running AD DS. They authenticate (issue Kerberos tickets), replicate, serve LDAP + DNS, provide Global Catalog, and act as time source (critical for Kerberos ±5 min skew).

**Multi-master replication** — all DCs are read-write; changes replicate everywhere. But some ops need a single owner — the **FSMO roles**:

| FSMO role | Scope | Controls |
|---|---|---|
| **Schema Master** | Forest | Schema updates |
| **Domain Naming Master** | Forest | Add/remove domains |
| **RID Master** | Domain | Hands out RID pools (SID = DomainSID + RID) |
| **PDC Emulator** | Domain | Time source, password-change priority, GPO master |
| **Infrastructure Master** | Domain | Cross-domain reference updates |

- **NTDS.dit** (`C:\Windows\NTDS\NTDS.dit`) — the AD database (ESE): all objects + **NTLM hashes** (incl. `krbtgt`), password history, group memberships.
- **SYSVOL** (`C:\Windows\SYSVOL`) — replicated share holding GPOs + logon scripts; **often leaks passwords** (GPP `cpassword`, scripts).
- **DRSUAPI** (MS-DRSR) — the DC-to-DC replication protocol abused by **DCSync**.

### Objects & key attributes
Everything is an object with attributes, defined by a class.

**User object attributes that matter:**
- `sAMAccountName` (legacy name), `userPrincipalName` (user@domain), `distinguishedName`, `objectSID` (never changes), `memberOf`, `servicePrincipalName` (if a service account), `passwordLastSet`.
- **`userAccountControl` (UAC) flags** — behavior bits attackers hunt:

| Flag (hex) | Name | Why it matters |
|---|---|---|
| `0x400000` | `DONT_REQ_PREAUTH` | **AS-REP Roastable** (no Kerberos pre-auth) |
| `0x80000` | `TRUSTED_FOR_DELEGATION` | **Unconstrained delegation** — can impersonate users |
| `0x0020` | `PASSWD_NOTREQD` | Account can have a blank password |
| `0x10000` | `DONT_EXPIRE_PASSWORD` | Password never expires |

**Computer objects** are security principals too — each domain machine has an account (name ends `$`) with a randomly-generated password rotated every ~30 days. DCs are in "Domain Controllers"; servers may register SPNs.

**Groups** — Security (permissions) vs Distribution (email only). Scopes: Domain Local, Global, Universal.

**Critical built-in groups:**

```text
Domain Admins      → full control of the domain, local admin on all domain hosts
Enterprise Admins  → full control of the forest (root domain only)
Schema Admins      → modify AD schema (should be empty)
Account Operators  → create/modify non-admin accounts
Backup Operators   → backup/restore any file; log on to DCs (privesc path)
DnsAdmins          → control DNS → code exec as SYSTEM on DC (exploit path)
```

### Forests, Trees & Trusts
Trusts define **authentication paths** (not authorization) between domains.

| Trust | Notes |
|---|---|
| Parent-Child | Auto, two-way transitive (`contoso.com` ⟷ `dev.contoso.com`) |
| Tree-Root | Auto, two-way transitive (new tree in forest) |
| External | Manual, cross-forest, non-transitive by default |
| Forest | Between forest roots, can be transitive |
| Realm | AD ↔ non-Windows Kerberos realm |

- **Direction:** "A trusts B" ⇒ users from **B** can access resources in **A** (A = trusting, B = trusted).
- **Transitive:** if A→B and B→C are transitive, A trusts C.
- **Security:** trusts are auth paths, not automatic permissions — but they're abused for lateral movement; **SID filtering** limits trust attacks. See [[Trusts & Persistence]].

---

## Authentication Concepts

**Authentication** ("who are you?") → proves identity, yields a token. **Authorization** ("what can you do?") → checks ACLs.

### NTLM (legacy, challenge-response)
```text
1. Client → Server: auth request (username)
2. Server → Client: random challenge
3. Client: encrypts challenge with NTLM hash → response
4. Server → DC: username + response
5. DC: computes same, compares → success/fail
```
Hash line format: `username:RID:LMhash:NTLMhash:::`
`Administrator:500:aad3b435...:31d6cfe0...:::`

**NTLM problems:** no mutual auth, relay-able, weak crypto, **hash == password (Pass-the-Hash)**, no replay protection. Still present for legacy/workgroup/fallback.

**NTLM vs NTLMv2:** NTLM = `MD4(password)` + DES challenge (no salt → precomputation, PtH, relay). NTLMv2 = `HMAC-MD5(user+domain+NTLM hash)` + client challenge + timestamp (stronger, replay-resistant, but **still relay-able**).

### Kerberos vs NTLM

| Feature | Kerberos | NTLM |
|---|---|---|
| Type | Ticket-based | Challenge-response |
| Mutual auth | Yes | No |
| Delegation | Yes | Limited |
| Encryption | AES, RC4 | MD4/MD5 |
| Performance | Better (cached tickets) | Slower (DC each auth) |
| Default since | Windows 2000 | Windows NT |
| Security | Stronger | Weaker |

### Security Identifiers (SID)
```text
S-1-5-21-<DomainID>-<RID>
   │ │  │      │        └ RID (unique in domain; 500=Administrator)
   │ │  │      └ domain identifier
   │ │  └ NT Authority (21)
   │ └ identifier authority (usually 5)
   └ revision (1)
```
**Well-known:** `-500` Administrator · `-512` Domain Admins · `-519` Enterprise Admins · `S-1-5-18` SYSTEM · `S-1-5-32-544` local Administrators. SIDs never change, drive ACLs, and appear in tokens.

---

## Kerberos Deep Dive

**Actors:** Principal (user, has password known to it + KDC), **KDC** on the DC = **AS** (issues TGTs) + **TGS** (issues service tickets), and the Service (has an account password, trusts KDC-issued tickets).

**Keys:** User key = `NTLM = MD4(password)`. **krbtgt key** encrypts all TGTs (→ Golden Ticket). Service account key encrypts service tickets (→ Silver Ticket). Session keys are per-session.

### The full flow
```text
USER            KDC(AS)         KDC(TGS)        SERVICE
 │──1 AS-REQ────►│               │               │   (username + timestamp enc w/ user key)
 │◄─2 AS-REP─────│               │               │   TGT (enc w/ krbtgt) + session key (enc w/ user key)
 │──3 TGS-REQ───────────────────►│               │   TGT + authenticator + target SPN
 │◄─4 TGS-REP────────────────────│               │   Service Ticket (enc w/ service key) + svc session key
 │──5 AP-REQ────────────────────────────────────►│   Service Ticket + authenticator
 │◄─6 AP-REP (optional)──────────────────────────│   mutual auth
```

- **AS-REP** encrypts the session key with the **user's password hash** → if **pre-auth is disabled**, anyone can request it and crack it offline = **AS-REP Roasting**.
- **TGS-REP** service ticket is encrypted with the **service account's hash** → any user can request it and crack it offline = **Kerberoasting**.

### Ticket structure & PAC
```text
TGT { enc w/ krbtgt hash: client, session key, start/end (10h), renew (7d), PAC }
Service Ticket { enc w/ service hash: client, svc session key, times, PAC }
```
**PAC** (Privilege Attribute Certificate) = Microsoft's authorization blob inside tickets: user SID, group SIDs (Domain Admins etc.), signed by KDC. Ticket files: **`.kirbi`** (Windows/Mimikatz) / **`.ccache`** (Linux) — convert with `ticketConverter.py`.

### Service Principal Names (SPN)
Format: `service_class/host:port` — e.g. `MSSQLSvc/sql.corp.local:1433`, `HTTP/web.corp.local`, `CIFS/fs.corp.local`, `LDAP/dc.corp.local`.
- Clients request tickets **by SPN**; any account with an SPN is **Kerberoastable**.
```powershell
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
setspn -Q */*
```

---

## Takeaways
- **Forest = security boundary; krbtgt = keys to the kingdom.**
- Attacks target the **ticket flow**: no-preauth → AS-REP roast; SPN → Kerberoast; krbtgt → Golden; service hash → Silver.
- **NTLM hash ≈ password** (PtH); **ticket ≈ identity** (PtT). Everything downstream builds on these.
- Next: [[AD Enumeration]] to find users, SPNs, no-preauth accounts, ACLs, and delegation.
