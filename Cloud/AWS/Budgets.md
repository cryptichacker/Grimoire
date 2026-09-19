---
tags: [cloud, aws, cost-management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Budgets

## Up
- [[AWS]]

**AWS Budgets** — Sets custom cost and usage budgets and alerts you when spend or utilization crosses defined thresholds.

**Category:** Cost Management

## Key concepts
- Cost, usage, RI/SP coverage and utilization budgets
- Threshold alerts (actual/forecast)
- Budget actions (auto-remediation)
- Notifications via SNS/email

## Common CLI
| Command | Description |
|---|---|
| `aws budgets create-budget` | Create a budget |
| `aws budgets describe-budgets` | List budgets |
| `aws budgets create-notification` | Add a threshold alert |

## Pricing model
First few budgets free; small per-budget-day charge beyond that.

## Works well with
- Cost Explorer
- SNS
- IAM (budget actions)

## Use cases
- Cost guardrails
- Spend alerts
- Automated cost controls
