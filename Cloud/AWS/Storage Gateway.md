---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Storage Gateway

## Up
- [[AWS]]

**AWS Storage Gateway** — Hybrid appliance that gives on-premises applications low-latency access to virtually unlimited AWS storage.

**Category:** Storage

## Key concepts
- S3 File Gateway (SMB/NFS to S3)
- FSx File Gateway
- Volume Gateway (iSCSI, cached or stored)
- Tape Gateway (virtual tape library)
- Local cache for hot data

## Common CLI
| Command | Description |
|---|---|
| `aws storagegateway list-gateways` | List gateways |
| `aws storagegateway create-nfs-file-share` | Create an NFS file share |

## Pricing model
Per gateway plus the underlying S3/EBS storage and data transfer.

## Works well with
- S3, S3 Glacier
- EBS, FSx
- AWS Backup

## Use cases
- Hybrid file storage
- Backup to cloud
- Physical tape replacement
