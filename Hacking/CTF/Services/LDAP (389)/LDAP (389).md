---
tags: [hacking, ctf, oscp, ldap, active-directory]
type: cheatsheet
source: compiled reference (LDAP enumeration)
last-verified: 2026-09-02
---

# LDAP (389)

## Up
- [[Services]]

LDAP (389, LDAPS 636; Global Catalog 3268) is the directory service — on Windows it's **Active Directory**. Wins: **anonymous binds** dumping users/groups, finding descriptions with passwords, and building user lists for AD attacks.

---

## Enumerate

```bash
nmap -p389 -sC -sV $IP
nmap --script "ldap*" -p389 $IP

# find the base DN
ldapsearch -x -H ldap://$IP -s base namingcontexts
#   → e.g. dc=corp,dc=local

# anonymous dump (if allowed)
ldapsearch -x -H ldap://$IP -b "dc=corp,dc=local"
ldapsearch -x -H ldap://$IP -b "dc=corp,dc=local" "(objectClass=person)"   # users
ldapsearch -x -H ldap://$IP -b "dc=corp,dc=local" | grep -i "description"   # creds in descriptions!

# with creds — richer dump
ldapdomaindump -u 'CORP\user' -p pass $IP -o ldapdump/
windapsearch -d corp.local --dc-ip $IP -u user -p pass -U    # users
```

---

## Foothold Techniques

- **Anonymous bind** → dump all users/groups → user list for spraying/roasting.
- **`description` / `info` fields** frequently contain **passwords** (classic).
- Identify **service accounts / SPNs**, admins, and group memberships → plan the AD path.
- Feed everything into [[Active Directory]] (Kerberoast/AS-REP/BloodHound).

---

## Common Misconfigurations

- **Anonymous/unauthenticated bind** allowed → full directory read.
- **Passwords in user attributes** (description, comment, userPassword).
- Verbose object data revealing the domain structure and privileged accounts.

---

## OSCP Tips

- LDAP(389) + Kerberos(88) + SMB(445) + DNS(53) = **Domain Controller** → this is an AD box.
- Always `grep -i description` on the dump — CTFs love hiding creds there.
- Build a **users.txt** from the dump for `kerbrute`, AS-REP roasting, and spraying — see [[Active Directory]].
- LDAPS(636)/GC(3268) are the same data on other ports — try each.
