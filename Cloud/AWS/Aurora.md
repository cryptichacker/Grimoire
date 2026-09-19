---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Aurora

## Up
- [[AWS]]

**Amazon Aurora** — AWS-built MySQL and PostgreSQL-compatible engine with a distributed storage layer, fast failover and up to 15 read replicas.

**Category:** Database

## Key concepts
- Cluster with a shared, auto-scaling storage volume
- Writer and reader endpoints
- Aurora Replicas for low-lag reads
- Serverless v2 (auto-scaling capacity units)
- Global Database for cross-region
- 6-way replication across 3 AZs; Backtrack (MySQL)

## Common CLI
| Command | Description |
|---|---|
| `aws rds create-db-cluster` | Create an Aurora cluster |
| `aws rds create-db-instance --engine aurora-mysql` | Add a cluster instance |
| `aws rds failover-db-cluster` | Trigger a failover |

## Pricing model
Per instance-hour (or ACU for Serverless v2) plus I/O and storage.

## Works well with
- RDS tooling, RDS Proxy
- Secrets Manager
- Lambda

## Use cases
- High-scale relational apps
- Multi-tenant SaaS databases
- Highly available databases
