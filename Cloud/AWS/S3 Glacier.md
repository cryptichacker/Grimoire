---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# S3 Glacier

## Up
- [[AWS]]

**Amazon S3 Glacier** — Archival S3 storage classes for rarely accessed data at very low cost, with configurable retrieval times.

**Category:** Storage

## Key concepts
- Glacier Instant Retrieval, Flexible Retrieval, Deep Archive
- Retrieval tiers: expedited, standard, bulk
- Vaults and the legacy Glacier API
- Vault Lock for WORM compliance
- Lifecycle transition from S3 Standard

## Common CLI
| Command | Description |
|---|---|
| `aws s3 cp ... --storage-class DEEP_ARCHIVE` | Write objects to an archive class |
| `aws glacier create-vault` | Create a vault (legacy API) |
| `aws glacier initiate-job` | Start an archive retrieval |

## Pricing model
Very low per GB-month storage; retrieval and request fees vary by tier.

## Works well with
- S3 lifecycle rules
- AWS Backup
- Athena (after restore)

## Use cases
- Compliance archives
- Long-term backups
- Cold/rarely accessed data
