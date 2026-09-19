---
tags: [cloud, aws, migration-transfer]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Database Migration Service

## Up
- [[AWS]]

**AWS Database Migration Service** — Migrates databases to AWS with minimal downtime, including homogeneous and heterogeneous (with schema conversion) migrations.

**Category:** Migration & Transfer

## Key concepts
- Replication instances
- Source and target endpoints
- Full-load and change data capture (CDC)
- Schema Conversion Tool (SCT/DMS Schema Conversion)
- Serverless option

## Common CLI
| Command | Description |
|---|---|
| `aws dms create-replication-instance` | Create a replication instance |
| `aws dms create-endpoint` | Define a source/target endpoint |
| `aws dms create-replication-task` | Create a migration task |
| `aws dms start-replication-task` | Run it |

## Pricing model
Per replication-instance-hour (or DCU serverless) plus storage.

## Works well with
- RDS/Aurora
- S3, Redshift
- Kinesis (CDC targets)

## Use cases
- Database migrations
- Engine conversions
- Ongoing replication to AWS
