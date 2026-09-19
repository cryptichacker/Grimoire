---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# RDS

## Up
- [[AWS]]

**Amazon Relational Database Service** — Managed relational databases that handle provisioning, patching, backups and failover for popular engines.

**Category:** Database

## Key concepts
- Engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- Multi-AZ standby for automatic failover
- Read replicas for scale-out reads
- Automated backups and manual snapshots
- Parameter and option groups
- Storage autoscaling; RDS Proxy for pooling

## Common CLI
| Command | Description |
|---|---|
| `aws rds create-db-instance` | Create a database |
| `aws rds describe-db-instances` | List databases |
| `aws rds create-db-snapshot` | Take a snapshot |
| `aws rds reboot-db-instance` | Reboot an instance |

## Pricing model
Per instance-hour plus storage, I/O and backup; Reserved Instances discount long-term use.

## Works well with
- EC2, VPC
- IAM, Secrets Manager
- CloudWatch

## Use cases
- Transactional applications
- Line-of-business databases
- Managed SQL without ops
