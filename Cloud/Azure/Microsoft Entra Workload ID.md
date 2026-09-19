---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Entra Workload ID

## Up
- [[Azure]]

**Microsoft Entra Workload ID** — Secures the identities used by apps, services and workloads (non-human identities) with governance and protection.

**Category:** Identity & Security

## Key concepts
- Workload identities (apps/service principals/managed identities)
- Federated identity credentials
- Conditional Access for workloads
- Access reviews for workloads
- Risk detection

## Common CLI
| Command | Description |
|---|---|
| `az ad sp create-for-rbac` | Create a workload (service principal) |
| `az identity federated-credential create` | Add a federated credential |
| `Portal: Workload ID` | Governance and protection |

## Pricing model
Per workload identity per month (premium).

## Works well with
- Managed Identities
- Conditional Access
- AKS / GitHub (federation)

## Use cases
- Securing non-human identities
- Credential-free federation
- Workload governance
