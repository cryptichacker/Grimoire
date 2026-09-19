---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Blob Storage

## Up
- [[Azure]]

**Azure Blob Storage** — Massively scalable object storage for unstructured data, with hot, cool, cold and archive tiers.

**Category:** Storage

## Key concepts
- Storage accounts
- Containers and blobs (block/append/page)
- Access tiers and lifecycle
- SAS tokens and RBAC
- Redundancy (LRS/ZRS/GRS)
- Versioning and immutability; static website hosting

## Common CLI
| Command | Description |
|---|---|
| `az storage account create` | Create a storage account |
| `az storage blob upload / download` | Move objects |
| `az storage container create` | Create a container |
| `az storage blob list` | List blobs |

## Pricing model
Per GB-month by tier plus transactions and egress.

## Works well with
- Data Lake Storage
- Front Door / CDN
- Functions (triggers), Synapse

## Use cases
- Backups and archives
- Data lakes
- Static sites and media
