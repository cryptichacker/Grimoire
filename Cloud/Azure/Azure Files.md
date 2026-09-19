---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Files

## Up
- [[Azure]]

**Azure Files** — Fully managed SMB and NFS file shares in the cloud, mountable from cloud and on-premises.

**Category:** Storage

## Key concepts
- File shares (SMB/NFS)
- Standard vs Premium
- Azure File Sync (hybrid)
- Snapshots
- Identity-based access (Entra/AD)
- Quotas

## Common CLI
| Command | Description |
|---|---|
| `az storage share-rm create` | Create a file share |
| `az storage file upload` | Upload a file |
| `az storage file list` | List files |
| `az storage share stats` | Share usage |

## Pricing model
Per GB provisioned (Premium) or used (Standard) plus transactions.

## Works well with
- VMs / AKS
- Azure File Sync
- Backup, Container Instances

## Use cases
- Shared file storage
- Lift-and-shift file shares
- Container persistence
