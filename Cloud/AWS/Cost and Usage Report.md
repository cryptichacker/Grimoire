---
tags: [cloud, aws, cost-management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cost and Usage Report

## Up
- [[AWS]]

**AWS Cost and Usage Report** — Delivers the most detailed, line-item cost and usage data to S3 for deep analysis (now via Data Exports).

**Category:** Cost Management

## Key concepts
- Hourly/daily/monthly granularity
- Resource-level line items
- Delivered to S3 (Parquet/CSV)
- Athena/Redshift/QuickSight integration
- Data Exports (new interface)

## Common CLI
| Command | Description |
|---|---|
| `aws cur put-report-definition` | Create a CUR definition |
| `aws cur describe-report-definitions` | List reports |
| `aws bcm-data-exports create-export` | Create a data export (new) |

## Pricing model
Free report; you pay for the S3 storage and any query engines used.

## Works well with
- S3
- Athena, Redshift
- QuickSight

## Use cases
- Detailed cost analysis
- Chargeback/showback
- Custom cost dashboards
