---
tags: [cloud, aws, cost-management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Savings Plans

## Up
- [[AWS]]

**AWS Savings Plans** — A flexible pricing model offering lower prices in exchange for a committed hourly spend over 1 or 3 years.

**Category:** Cost Management

## Key concepts
- Compute vs EC2 Instance vs SageMaker Savings Plans
- 1- or 3-year terms
- No/partial/all upfront
- Applies automatically across usage
- Utilization and coverage reports

## Common CLI
| Command | Description |
|---|---|
| `aws savingsplans describe-savings-plans-offerings` | Browse offerings |
| `aws savingsplans create-savings-plan` | Purchase a plan |
| `aws savingsplans describe-savings-plans` | List your plans |

## Pricing model
Committed hourly spend; discounts vary by plan type and term.

## Works well with
- Cost Explorer
- EC2, Fargate, Lambda
- SageMaker

## Use cases
- Compute cost savings
- Predictable-workload discounts
- Budget commitment
