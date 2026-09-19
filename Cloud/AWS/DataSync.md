---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# DataSync

## Up
- [[AWS]]

**AWS DataSync** — Online data-transfer service that moves large amounts of data to, from and between AWS storage services quickly.

**Category:** Storage

## Key concepts
- Agents for on-prem sources
- Locations (source/destination)
- Tasks and task executions
- Scheduling and bandwidth throttling
- In-transit encryption and verification

## Common CLI
| Command | Description |
|---|---|
| `aws datasync create-location-s3` | Define an S3 location |
| `aws datasync create-task` | Create a transfer task |
| `aws datasync start-task-execution` | Run a transfer |

## Pricing model
Per GB of data transferred.

## Works well with
- S3, EFS, FSx
- Storage Gateway
- CloudWatch

## Use cases
- Migrations
- Recurring data replication
- Cross-storage transfers
