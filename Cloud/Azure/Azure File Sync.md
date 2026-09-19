---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure File Sync

## Up
- [[Azure]]

**Azure File Sync** — Centralizes on-premises file shares in Azure Files while keeping the flexibility and performance of a local cache.

**Category:** Storage

## Key concepts
- Storage Sync Service
- Sync groups (cloud + server endpoints)
- Cloud tiering
- Multi-site sync
- Fast disaster recovery

## Common CLI
| Command | Description |
|---|---|
| `az storagesync create` | Create a Storage Sync Service |
| `az storagesync sync-group create` | Create a sync group |
| `az storagesync sync-group cloud-endpoint create` | Add a cloud endpoint |

## Pricing model
Per server registered plus underlying Azure Files storage.

## Works well with
- Azure Files
- Windows Server
- Azure Backup

## Use cases
- Hybrid file shares
- Branch-office caching
- File-server consolidation
