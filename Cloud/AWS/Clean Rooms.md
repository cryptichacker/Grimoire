---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Clean Rooms

## Up
- [[AWS]]

**AWS Clean Rooms** — Lets companies securely collaborate and analyze combined datasets without sharing or copying the raw underlying data.

**Category:** Analytics

## Key concepts
- Collaborations and members
- Configured tables
- Analysis rules (aggregation/list/custom)
- Differential privacy option
- Cryptographic computing

## Common CLI
| Command | Description |
|---|---|
| `aws cleanrooms create-collaboration` | Create a collaboration |
| `aws cleanrooms create-configured-table` | Configure a table |
| `aws cleanrooms start-protected-query` | Run a protected query |

## Pricing model
Per Clean Rooms Processing Unit (CRPU) used by queries.

## Works well with
- S3, Glue
- Athena
- QuickSight

## Use cases
- Privacy-safe data collaboration
- Advertising measurement
- Cross-company analytics
