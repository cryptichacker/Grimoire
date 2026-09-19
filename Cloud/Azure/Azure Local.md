---
tags: [cloud, azure, hybrid]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Local

## Up
- [[Azure]]

**Azure Local** — A hyperconverged infrastructure (HCI) solution (formerly Azure Stack HCI) that runs Azure services in your datacenter, managed via Arc.

**Category:** Hybrid

## Key concepts
- HCI clusters on validated hardware
- Runs VMs, AKS and Arc services on-prem
- Managed through Azure/Arc
- Storage Spaces Direct
- Cloud-based monitoring and updates

## Common CLI
| Command | Description |
|---|---|
| `az stack-hci cluster create` | Create/register a cluster |
| `az stack-hci cluster list` | List clusters |
| `az stack-hci-vm create` | Create a VM on the cluster |

## Pricing model
Per physical core per month subscription.

## Works well with
- Azure Arc
- AKS (on-prem)
- Azure Monitor / Backup

## Use cases
- Hybrid infrastructure
- Edge datacenters
- On-prem Azure services
