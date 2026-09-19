---
tags: [cloud, azure, migration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Data Box

## Up
- [[Azure]]

**Azure Data Box** — A family of physical devices to transfer large amounts of data into and out of Azure when networks are impractical.

**Category:** Migration

## Key concepts
- Data Box (80 TB), Data Box Disk, Data Box Heavy
- Ordered and shipped by Microsoft
- Encrypted at rest
- Offline bulk transfer
- Copy via SMB/NFS/REST

## Common CLI
| Command | Description |
|---|---|
| `az databox job create` | Create a Data Box order |
| `az databox job list` | List orders |
| `az databox job show` | Show order status |

## Pricing model
Per-device order fee plus shipping (data transfer into Azure is free).

## Works well with
- Blob Storage
- Azure Files
- Azure Migrate

## Use cases
- Large offline migrations
- Bandwidth-limited transfers
- Bulk data seeding
