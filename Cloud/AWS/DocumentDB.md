---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# DocumentDB

## Up
- [[AWS]]

**Amazon DocumentDB** — Managed document database compatible with the MongoDB API, with storage that scales automatically.

**Category:** Database

## Key concepts
- Cluster separates compute from storage
- Replica instances for read scaling
- MongoDB-compatible API
- 6-way replicated storage across AZs
- Backups and point-in-time recovery
- VPC-only access

## Common CLI
| Command | Description |
|---|---|
| `aws docdb create-db-cluster` | Create a cluster |
| `aws docdb create-db-instance` | Add an instance |
| `aws docdb describe-db-clusters` | List clusters |

## Pricing model
Per instance-hour plus storage, I/O and backup.

## Works well with
- VPC, KMS
- Secrets Manager
- CloudWatch

## Use cases
- JSON/document workloads
- MongoDB migrations
- Content and catalog data
