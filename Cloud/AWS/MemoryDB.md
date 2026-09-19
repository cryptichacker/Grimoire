---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MemoryDB

## Up
- [[AWS]]

**Amazon MemoryDB** — Redis/Valkey-compatible, durable in-memory database offering microsecond reads and Multi-AZ durability.

**Category:** Database

## Key concepts
- Durable in-memory via multi-AZ transaction log
- Redis/Valkey-compatible API
- Clusters with shards and replicas
- Snapshots
- ACLs and encryption

## Common CLI
| Command | Description |
|---|---|
| `aws memorydb create-cluster` | Create a cluster |
| `aws memorydb describe-clusters` | List clusters |
| `aws memorydb create-snapshot` | Create a snapshot |

## Pricing model
Per node-hour plus data written and snapshot storage.

## Works well with
- VPC
- KMS
- CloudWatch

## Use cases
- Primary in-memory database
- Durable caches
- Session and leaderboard data
