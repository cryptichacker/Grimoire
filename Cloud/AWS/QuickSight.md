---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# QuickSight

## Up
- [[AWS]]

**Amazon QuickSight** — Cloud-native, serverless business-intelligence service for interactive dashboards and ML-powered insights.

**Category:** Analytics

## Key concepts
- Data sources and datasets
- SPICE in-memory engine
- Analyses and dashboards
- ML Insights and Q (natural language)
- Embedded analytics
- Row-level security

## Common CLI
| Command | Description |
|---|---|
| `aws quicksight create-data-set` | Create a dataset |
| `aws quicksight create-dashboard` | Create a dashboard |
| `aws quicksight list-dashboards` | List dashboards |

## Pricing model
Per author and reader (subscription) plus SPICE capacity.

## Works well with
- Redshift, Athena
- RDS
- S3

## Use cases
- BI dashboards
- Embedded analytics
- Self-service reporting
