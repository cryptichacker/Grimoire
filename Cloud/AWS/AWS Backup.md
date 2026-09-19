---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AWS Backup

## Up
- [[AWS]]

**AWS Backup** — Centralized service that automates and governs backups across many AWS services from a single place.

**Category:** Storage

## Key concepts
- Backup plans (schedules and rules)
- Backup vaults
- Lifecycle to cold storage
- Cross-region and cross-account copy
- Vault Lock for immutability
- Resource assignment by tag

## Common CLI
| Command | Description |
|---|---|
| `aws backup create-backup-plan` | Define a backup plan |
| `aws backup start-backup-job` | Run a backup on demand |
| `aws backup list-recovery-points-by-backup-vault` | List restore points |

## Pricing model
Per GB of backup storage plus restore and cross-region copy charges.

## Works well with
- EBS, RDS, DynamoDB
- EFS, S3
- Storage Gateway

## Use cases
- Centralized backup policy
- Compliance retention
- Disaster-recovery copies
