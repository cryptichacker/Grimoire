---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# ElastiCache

## Up
- [[AWS]]

**Amazon ElastiCache** — Managed in-memory data store using Redis/Valkey or Memcached for microsecond-latency caching.

**Category:** Database

## Key concepts
- Redis/Valkey vs Memcached
- Cluster mode (sharding)
- Replication groups and automatic failover
- Node types and sizing
- Backups and encryption/AUTH (Redis)
- Serverless option

## Common CLI
| Command | Description |
|---|---|
| `aws elasticache create-cache-cluster` | Create a Memcached or single-node cluster |
| `aws elasticache create-replication-group` | Create Redis with replicas |
| `aws elasticache describe-cache-clusters` | List clusters |

## Pricing model
Per node-hour (plus backup storage); Serverless billed by usage.

## Works well with
- EC2
- RDS / Aurora
- Lambda, VPC

## Use cases
- Application caching
- Session stores
- Leaderboards and rate limiting
