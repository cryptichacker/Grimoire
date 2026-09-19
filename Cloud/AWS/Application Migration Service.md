---
tags: [cloud, aws, migration-transfer]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Application Migration Service

## Up
- [[AWS]]

**AWS Application Migration Service** — Lift-and-shift (rehost) service that replicates on-prem or cloud servers to AWS with minimal downtime.

**Category:** Migration & Transfer

## Key concepts
- Continuous block-level replication
- Test and cutover instances
- Agent-based source servers (MGN)
- Post-launch actions
- Wave planning

## Common CLI
| Command | Description |
|---|---|
| `aws mgn describe-source-servers` | List source servers |
| `aws mgn start-replication` | Start replication |
| `aws mgn start-cutover` | Cut over to AWS |

## Pricing model
Free for a period per server during migration; pay for staging and target EC2/EBS.

## Works well with
- EC2, EBS
- Migration Hub
- VPC

## Use cases
- Lift-and-shift migrations
- Data-center exit
- Cross-cloud rehosting
