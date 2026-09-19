---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Data Box Gateway

## Up
- [[Azure]]

**Azure Data Box Gateway** — A virtual appliance that sends data to Azure Storage over the network, presenting local SMB/NFS shares.

**Category:** Storage

## Key concepts
- Virtual storage gateway
- Local SMB/NFS shares
- Automatic upload to Blob/Files
- Bandwidth throttling
- Cloud tiering

## Common CLI
| Command | Description |
|---|---|
| `az databoxedge device list` | List gateway/edge devices |
| `az databoxedge share create` | Create a share |
| `az databoxedge device show` | Show device details |

## Pricing model
Per gateway plus underlying Azure Storage and egress.

## Works well with
- Blob Storage / Azure Files
- Storage Mover
- On-prem apps

## Use cases
- Continuous data ingestion
- Cloud-tiered file shares
- Network data transfer
