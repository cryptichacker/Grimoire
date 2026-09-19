---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EC2 Auto Scaling

## Up
- [[AWS]]

**Amazon EC2 Auto Scaling** — Automatically launches or terminates EC2 instances to maintain availability and match demand.

**Category:** Compute

## Key concepts
- Auto Scaling Groups
- Launch templates
- Desired/min/max capacity
- Scaling policies (target tracking, step, scheduled)
- Health checks and instance refresh
- Mixed instances and Spot

## Common CLI
| Command | Description |
|---|---|
| `aws autoscaling create-auto-scaling-group` | Create an ASG |
| `aws autoscaling update-auto-scaling-group` | Change capacity/config |
| `aws autoscaling set-desired-capacity` | Scale now |
| `aws autoscaling describe-auto-scaling-groups` | List ASGs |

## Pricing model
Free; pay for the EC2 instances launched.

## Works well with
- EC2
- Elastic Load Balancing
- CloudWatch

## Use cases
- Elastic web tiers
- Self-healing fleets
- Cost-optimized scaling
