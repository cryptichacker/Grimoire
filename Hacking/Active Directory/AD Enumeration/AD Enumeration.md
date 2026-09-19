---
tags: [hacking, active-directory, windows, enumeration]
type: cheatsheet
source: personal Notion — "Active Directory" cheatsheet + additions
last-verified: 2026-09-03
---

# AD Enumeration

## Up
- [[Active Directory]]

Gather domain, user, group, computer, SPN, and ACL information — the first step after (and often before) getting credentials. Enumerate thoroughly; the path is almost always in the data. Note: modern **CrackMapExec → NetExec (`nxc`)** — same syntax.

---

## Network / DC Discovery

```bash
# AD service sweep
nmap -Pn -p 53,88,135,139,389,445,464,636,3268,3269,3389,5985 <target-ip>
nmap -sV -sC -p- -T4 <target-ip> -oA ad_scan

# confirm a Domain Controller
nmap -p389 --script ldap-rootdse <target-ip>     # leaks domain/forest DN, DC name
```

DNS(53)+Kerberos(88)+LDAP(389)+SMB(445) ⇒ Domain Controller. Add the domain/DC hostnames to `/etc/hosts`.

## DNS
```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>     # find DCs
nslookup -type=SRV _kerberos._tcp.<domain>
dnsenum <domain>
dig axfr @<dns-server> <domain>                       # zone transfer (usually denied)
```

---

## SMB Enumeration

```bash
# enum4linux(-ng) — the classic all-in-one
enum4linux-ng -A <target-ip>
enum4linux -a <target-ip>            # -U users, -S shares, -G groups, -P pass-pol, -r RID cycling

# smbclient / smbmap
smbclient -L //<ip>/ -N               # null-session share list
smbclient //<ip>/share -U user
smbmap -H <ip> -u user -p pass        # shows READ/WRITE per share
smbmap -H <ip> -u null                # null

# NetExec / CrackMapExec — the swiss army knife
nxc smb <ip>                          # OS, domain, signing (signing:False = relay target)
nxc smb <ip> -u '' -p '' --shares     # null session
nxc smb <ip> -u user -p pass --users --groups --shares --pass-pol
nxc smb <ip> -u user -p pass --rid-brute     # enumerate objects by RID
nxc smb <ip> -u user -H <ntlm> --shares      # pass-the-hash
```

Reveals: domain/workgroup, domain SID, users (sometimes unauth), groups, **password policy** (spray safely!), shares, OS version.

---

## LDAP Enumeration

```bash
# base / naming contexts (anonymous bind check)
ldapsearch -x -H ldap://<ip> -s base namingcontexts

# dump users / groups
ldapsearch -x -H ldap://<ip> -b "DC=corp,DC=local" "(objectClass=user)"
ldapsearch -x -H ldap://<ip> -D "user@corp.local" -w pass -b "DC=corp,DC=local" "(objectClass=user)"

# Kerberoastable (SPN set)
ldapsearch ... "(&(objectClass=user)(servicePrincipalName=*))"
# AS-REP roastable (no preauth: UAC bit 0x400000 = 4194304)
ldapsearch ... "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))"
# admins / protected accounts
ldapsearch ... "(adminCount=1)"

# python tooling
windapsearch.py -d corp.local --dc-ip <ip> -u user -p pass --users --computers --privileged-users
ldapdomaindump -u 'CORP\user' -p pass <ip> -o ldapdump/     # HTML/JSON of the whole directory
nxc ldap <ip> -u user -p pass --asreproast asrep.txt --kerberoasting kerb.txt   # one-shot roast lists
```

**Filter cheatsheet:** `(objectClass=user|computer|group)` · `(sAMAccountName=alice)` · `(servicePrincipalName=*)` · `(adminCount=1)` · `(&(a)(b))` AND · `(|(a)(b))` OR · UAC bit match `userAccountControl:1.2.840.113556.1.4.803:=<flag>`.

---

## Kerberos User Enumeration (Kerbrute)

Validates usernames via AS-REQ **without logging failed logons or triggering lockout**.

```bash
kerbrute userenum --dc <dc-ip> -d <domain> users.txt -o valid_users.txt
kerbrute passwordspray --dc <dc-ip> -d <domain> valid_users.txt 'Winter2026!'
kerbrute bruteuser --dc <dc-ip> -d <domain> passwords.txt username   # single user, cautious
```
Wordlists: `/usr/share/seclists/Usernames/…` + OSINT-derived names ([[OSINT]]/[[theHarvester]]).

## RPC
```bash
rpcclient -U "" -N <ip>              # null session
rpcclient -U "user%pass" <ip>
  > enumdomusers        # users
  > enumdomgroups       # groups
  > querygroupmem <RID> # members
  > getdompwinfo        # password policy
  > lookupnames <user>  # → SID
impacket-samrdump <domain>/<user>:<pass>@<ip>
```

---

## BloodHound — the map

Collect the directory into a graph, then let it compute the shortest path to Domain Admin. **Do this the moment you have any creds.**

```bash
# Linux collector
bloodhound-python -u user -p pass -ns <dc-ip> -d corp.local -c All
bloodhound-python -u user --hashes :<ntlm> -ns <dc-ip> -d corp.local -c All
# NetExec collector
nxc ldap <ip> -u user -p pass --bloodhound --collection All --dns-server <dc-ip>

# Windows collector (SharpHound)
.\SharpHound.exe -c All                 # or --Stealth ; -c DCOnly (quiet, no sessions)

# view: start neo4j + BloodHound GUI, drag-drop the ZIP
sudo neo4j console          # http://localhost:7474  (neo4j/neo4j)
bloodhound
```

**Key pre-built queries:** Shortest Paths to Domain Admins · Kerberoastable users · AS-REP roastable users · Principals with DCSync rights · Computers with Unconstrained Delegation · Computers where Domain Users are local admin.

```cypher
// shortest path any owned user → Domain Admins
MATCH p=shortestPath((u:User)-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})) RETURN p
// unconstrained delegation
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c.name
// DCSync rights
MATCH p=(u)-[:MemberOf|GetChanges|GetChangesAll*1..]->(d:Domain) RETURN u.name
```

Mark nodes you own as **Owned**, then use "Shortest paths from Owned principals."

---

## Password Policy (always get it before spraying)
```bash
nxc smb <dc-ip> --pass-pol
enum4linux -P <ip>
rpcclient -U "" -N <ip> -c getdompwinfo
```
Note lockout threshold/window → spray fewer attempts than the threshold, spaced over the window. See spraying in [[Kerberos Attacks]].

---

## Takeaways
- **Unauth first:** null sessions, RID cycling, Kerbrute userenum, anon LDAP — build a user list without creds.
- **First creds → BloodHound + roast lists** (`--asreproast`, `--kerberoasting`) in one pass.
- Grab the **password policy** before any spray; harvest **SPNs, no-preauth accounts, admins, and delegation** for the attacks in [[Kerberos Attacks]], [[Delegation Attacks]], [[ACL & GPO Abuse]].
