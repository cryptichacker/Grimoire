---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Detective

## Up
- [[AWS]]

**Amazon Detective** — Analyzes and visualizes security data to help investigate the root cause and scope of potential security issues.

**Category:** Security

## Key concepts
- Behavior graph
- Automatic ingestion (VPC Flow, CloudTrail, GuardDuty)
- Entity profiles and finding groups
- Interactive investigation
- Time-range analysis

## Common CLI
| Command | Description |
|---|---|
| `aws detective create-graph` | Create a behavior graph |
| `aws detective list-graphs` | List graphs |
| `aws detective list-members` | List member accounts |

## Pricing model
Per GB of data ingested and analyzed.

## Works well with
- GuardDuty, Security Hub
- CloudTrail
- VPC

## Use cases
- Security investigations
- Root-cause analysis
- Threat hunting
