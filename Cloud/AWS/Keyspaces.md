---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Keyspaces

## Up
- [[AWS]]

**Amazon Keyspaces** — Serverless, Apache Cassandra-compatible database that scales tables automatically with no clusters to manage.

**Category:** Database

## Key concepts
- Cassandra Query Language (CQL)
- Keyspaces and tables
- On-demand vs provisioned capacity
- Point-in-time recovery
- Encryption and VPC endpoints

## Common CLI
| Command | Description |
|---|---|
| `aws keyspaces create-keyspace` | Create a keyspace |
| `aws keyspaces create-table` | Create a table |
| `aws keyspaces list-keyspaces` | List keyspaces |

## Pricing model
Per read/write request (on-demand) or provisioned capacity plus storage.

## Works well with
- VPC
- KMS
- CloudWatch, CloudTrail

## Use cases
- Cassandra migrations
- High-scale wide-column data
- IoT and time-series apps
