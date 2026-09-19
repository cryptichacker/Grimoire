---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# FinSpace

## Up
- [[AWS]]

**Amazon FinSpace** — A data-management and analytics service purpose-built for the financial-services industry, with time-series tooling.

**Category:** Analytics

## Key concepts
- Managed kdb Insights (time-series)
- Data catalog and datasets
- Analytics environments
- Permission groups
- Bi-temporal data

## Common CLI
| Command | Description |
|---|---|
| `aws finspace create-environment` | Create an environment |
| `aws finspace list-environments` | List environments |
| `aws finspace-data create-dataset` | Create a dataset |

## Pricing model
Per environment plus compute and storage used.

## Works well with
- S3
- kdb / managed clusters
- QuickSight

## Use cases
- Financial time-series analytics
- Quant research
- Market data management
