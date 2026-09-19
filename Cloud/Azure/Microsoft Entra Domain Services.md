---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Entra Domain Services

## Up
- [[Azure]]

**Microsoft Entra Domain Services** — Provides managed domain services (domain join, LDAP, Kerberos/NTLM, Group Policy) without deploying domain controllers.

**Category:** Identity & Security

## Key concepts
- Managed domain
- Domain join for VMs
- LDAP and Kerberos/NTLM
- Group Policy
- One-way sync from Entra ID
- No DC management

## Common CLI
| Command | Description |
|---|---|
| `Portal/ARM: create managed domain` | Provision Entra Domain Services |
| `az resource create` | Create via ARM template |

## Pricing model
Per managed domain-hour by SKU.

## Works well with
- Microsoft Entra ID
- Virtual Machines
- Legacy AD-integrated apps

## Use cases
- Lift-and-shift AD apps
- Domain join in the cloud
- LDAP/Kerberos without DCs
