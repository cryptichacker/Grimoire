---
tags: [cloud, aws, migration-transfer]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Elastic Disaster Recovery

## Up
- [[AWS]]

**AWS Elastic Disaster Recovery** — Scalable disaster recovery (DRS) that continuously replicates servers to AWS for fast, low-cost failover.

**Category:** Migration & Transfer

## Key concepts
- Continuous block-level replication
- Low-cost staging area
- Point-in-time recovery
- Automated failover and failback
- Drills without disruption

## Common CLI
| Command | Description |
|---|---|
| `aws drs describe-source-servers` | List replicated servers |
| `aws drs start-recovery` | Launch recovery instances |

## Pricing model
Per source server plus low-cost staging storage; full instances only during drills/failover.

## Works well with
- EC2, EBS
- VPC
- CloudWatch

## Use cases
- Disaster recovery
- Cross-Region failover
- On-prem to AWS DR
