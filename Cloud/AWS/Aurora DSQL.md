---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Aurora DSQL

## Up
- [[AWS]]

**Amazon Aurora DSQL** — Serverless, distributed SQL database with active-active multi-Region strong consistency and virtually unlimited scale.

**Category:** Database

## Key concepts
- PostgreSQL-compatible
- Serverless, auto-scaling
- Multi-Region active-active
- Strong consistency
- No infrastructure to manage

## Common CLI
| Command | Description |
|---|---|
| `aws dsql create-cluster` | Create a cluster |
| `aws dsql get-cluster` | Get cluster details |
| `aws dsql list-clusters` | List clusters |

## Pricing model
Per request/compute unit and storage consumed (serverless).

## Works well with
- IAM (auth)
- VPC
- Lambda

## Use cases
- Globally distributed apps
- High-availability SQL
- Multi-Region writes
