---
tags: [hacking, ctf, oscp, services, enumeration]
type: moc
---

# Services

## Up
- [[CTF]]

Per-port playbooks — for each open port: **enumeration → foothold techniques → common misconfigurations**. This is the core of a CTF: after [[Recon & Enumeration]] gives you the open ports, jump to the matching note.

## Service Notes (common OSCP ports)
- [[FTP (21)]] — anon login, file up/down, version exploits
- [[SSH (22)]] — creds/brute, key reuse, version bugs
- [[SMTP & Mail (25,110,143)]] — user enum (VRFY), reading mail
- [[DNS (53)]] — zone transfers, subdomain/vhost discovery
- [[HTTP (80,443)]] — the biggest attack surface: dirbust, vhosts, CMS, LFI/RCE/upload
- [[SMB (139,445)]] — null sessions, shares, RCE (EternalBlue), creds
- [[SNMP (161)]] — community strings, system/user/process leak
- [[LDAP (389)]] — directory dumps, AD user enum
- [[MSSQL (1433)]] — xp_cmdshell, `sa` creds, impersonation
- [[NFS (2049)]] — exports, no_root_squash privesc
- [[MySQL (3306)]] — creds, UDF/`INTO OUTFILE`, cred reuse
- [[RDP (3389)]] — creds/brute, session access
- [[PostgreSQL (5432)]] — creds, `COPY ... PROGRAM` RCE
- [[WinRM (5985,5986)]] — evil-winrm with creds/hashes
- [[Redis (6379)]] — unauth access, webshell/SSH-key write

---

## Long-Tail Ports Quick Reference

The services without a dedicated note yet — enough to get started:

| Port | Service | First moves |
|---|---|---|
| 23 | **Telnet** | Banner, default/weak creds, cleartext creds on the wire |
| 69/udp | **TFTP** | No auth — `get`/`put` files (grab configs, drop webshell) |
| 79 | **Finger** | User enumeration (`finger @host`) |
| 88 | **Kerberos** | AD → user enum (kerbrute), AS-REP roast, Kerberoast → see [[Active Directory]] |
| 111 | **RPCbind** | `rpcinfo -p $IP`; leads to NFS/other RPC services |
| 135 | **MSRPC** | `rpcdump`, `impacket` DCOM; Windows enumeration |
| 512-514 | **r-services** | rlogin/rsh trust abuse, `.rhosts` |
| 873 | **rsync** | `rsync -av rsync://$IP/` list modules; read/write files |
| 1099 | **Java RMI** | `ysoserial` deserialization RCE |
| 2375 | **Docker API** | Unauth → mount host, container escape → root |
| 3128 | **Squid proxy** | Pivot to internal ports |
| 5900 | **VNC** | Weak/no password (`vncviewer`), password crack |
| 6000 | **X11** | Screenshot/keylog if open (`xspy`, `xwd`) |
| 9200 | **Elasticsearch** | Unauth REST API, CVE-2015-1427 RCE on old versions |
| 11211 | **Memcached** | `stats`, dump keys (unauth) |
| 27017 | **MongoDB** | Unauth `mongo`, dump collections for creds |
| 8080/8000/8443 | **Alt HTTP** | Treat like [[HTTP (80,443)]]; Tomcat/Jenkins/etc. |

For any of these: **banner + version → searchsploit → default creds → misconfig/anon access**, same loop as [[Recon & Enumeration]].

---

## Universal Per-Service Checklist

For every service you enumerate, ask:

1. **Version** → `searchsploit`, CVE lookup.
2. **Anonymous / unauthenticated** access? (FTP anon, SMB null, SNMP public, Redis/Mongo/Elastic unauth).
3. **Default / weak creds**? Then targeted [[Hydra]].
4. **Config / file disclosure** → creds, hostnames, keys, other ports.
5. **Credential reuse** — anything you find, try **everywhere** ([[Password Cracking]]).
6. **Does it hint at another service/vhost/user?** Feed it back into recon.
