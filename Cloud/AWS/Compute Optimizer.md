---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Compute Optimizer

## Up
- [[AWS]]

**AWS Compute Optimizer** — Analyzes utilization metrics and recommends optimal, right-sized resource configurations to cut cost and improve performance.

**Category:** Management

## Key concepts
- Recommendations for EC2, Auto Scaling groups, EBS, Lambda and ECS/Fargate
- Utilization analysis
- Rightsizing and savings estimates
- Enhanced infrastructure metrics

## Common CLI
| Command | Description |
|---|---|
| `aws compute-optimizer get-ec2-instance-recommendations` | EC2 rightsizing recommendations |
| `aws compute-optimizer get-recommendation-summaries` | Recommendation summaries |

## Pricing model
Free for basic recommendations; enhanced metrics cost extra.

## Works well with
- EC2, Auto Scaling
- EBS, Lambda
- CloudWatch

## Use cases
- Rightsizing
- Cost optimization
- Performance tuning
