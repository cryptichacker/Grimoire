---
tags: [cloud, aws, cost-management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cost Explorer

## Up
- [[AWS]]

**AWS Cost Explorer** — Visualizes, understands and forecasts your AWS spend and usage over time with filtering and grouping.

**Category:** Cost Management

## Key concepts
- Cost and usage reports/graphs
- Filtering and grouping by dimension/tag
- Forecasting
- Rightsizing and reservation recommendations
- Savings Plans utilization

## Common CLI
| Command | Description |
|---|---|
| `aws ce get-cost-and-usage` | Query cost and usage |
| `aws ce get-cost-forecast` | Forecast spend |
| `aws ce get-rightsizing-recommendation` | Rightsizing suggestions |

## Pricing model
Console UI free; per paginated API request charged.

## Works well with
- Budgets
- Cost and Usage Report
- Compute Optimizer

## Use cases
- Spend analysis
- Cost forecasting
- Savings recommendations
