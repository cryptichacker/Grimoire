---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Storage Mover

## Up
- [[Azure]]

**Azure Storage Mover** — A managed migration service for moving files and folders from on-prem or other clouds into Azure Storage.

**Category:** Storage

## Key concepts
- Storage Mover resource and agents
- Projects and job definitions
- SMB and NFS + S3 sources
- Migration monitoring
- Bandwidth control

## Common CLI
| Command | Description |
|---|---|
| `az storage-mover create` | Create a Storage Mover |
| `az storage-mover project create` | Create a project |
| `az storage-mover job-definition create` | Define a migration job |

## Pricing model
Free service; pay for the target storage and any egress.

## Works well with
- Blob Storage / Azure Files
- DataSync-style agents
- Azure Migrate

## Use cases
- File-share migration
- Cloud-to-cloud storage moves
- Recurring data sync
