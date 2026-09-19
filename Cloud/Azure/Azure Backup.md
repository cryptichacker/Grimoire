---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Backup

## Up
- [[Azure]]

**Azure Backup** — A centralized, managed backup service protecting VMs, databases, files and on-prem workloads.

**Category:** Storage

## Key concepts
- Recovery Services vault / Backup vault
- Backup policies and retention
- VM, SQL, SAP HANA, Files, Blob backup
- Cross-region restore
- Soft delete and immutable vaults

## Common CLI
| Command | Description |
|---|---|
| `az backup vault create` | Create a Recovery Services vault |
| `az backup protection enable-for-vm` | Protect a VM |
| `az backup job list` | List backup jobs |

## Pricing model
Per protected instance plus backup storage consumed.

## Works well with
- Virtual Machines
- Azure SQL / Files
- Site Recovery

## Use cases
- Centralized backup
- Ransomware protection
- Compliance retention
