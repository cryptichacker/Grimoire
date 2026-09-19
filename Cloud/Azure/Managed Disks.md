---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Managed Disks

## Up
- [[Azure]]

**Azure Managed Disks** — Block-level storage volumes managed by Azure for use with VMs, in several performance tiers.

**Category:** Storage

## Key concepts
- Disk types (Ultra, Premium SSD v2/v1, Standard SSD/HDD)
- OS vs data disks
- Snapshots and images
- Encryption (SSE / Azure Disk Encryption)
- Shared disks
- Bursting

## Common CLI
| Command | Description |
|---|---|
| `az disk create` | Create a disk |
| `az vm disk attach` | Attach to a VM |
| `az snapshot create` | Snapshot a disk |
| `az disk list` | List disks |

## Pricing model
Per provisioned GB by tier (plus IOPS/throughput for some).

## Works well with
- Virtual Machines / Scale Sets
- Backup
- Key Vault

## Use cases
- VM boot/data disks
- Databases on VMs
- Persistent app data
