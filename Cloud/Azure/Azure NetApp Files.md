---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure NetApp Files

## Up
- [[Azure]]

**Azure NetApp Files** — Enterprise-grade, high-performance file storage powered by NetApp ONTAP, for demanding NFS/SMB workloads.

**Category:** Storage

## Key concepts
- Capacity pools and volumes
- Service levels (Standard/Premium/Ultra)
- NFS and SMB protocols
- Snapshots and cross-region replication
- Application volume groups (SAP HANA)

## Common CLI
| Command | Description |
|---|---|
| `az netappfiles account create` | Create a NetApp account |
| `az netappfiles pool create` | Create a capacity pool |
| `az netappfiles volume create` | Create a volume |

## Pricing model
Per provisioned GB by service level.

## Works well with
- VMs / AKS
- SAP on Azure
- HPC workloads

## Use cases
- High-performance NAS
- SAP HANA storage
- Enterprise file shares
