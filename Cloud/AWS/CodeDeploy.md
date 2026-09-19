---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CodeDeploy

## Up
- [[AWS]]

**AWS CodeDeploy** — Automates application deployments to EC2, on-prem, ECS and Lambda with controlled rollouts and rollback.

**Category:** Developer Tools

## Key concepts
- Applications and deployment groups
- Deployment configs (all-at-once/rolling/canary/blue-green)
- appspec file
- Lifecycle hooks
- Automatic rollback on alarm

## Common CLI
| Command | Description |
|---|---|
| `aws deploy create-application` | Create an application |
| `aws deploy create-deployment` | Start a deployment |
| `aws deploy get-deployment` | Check status |

## Pricing model
Free to EC2/Lambda/ECS you own; charged per on-prem instance update.

## Works well with
- CodePipeline
- EC2 / Auto Scaling
- ECS, Lambda

## Use cases
- Zero-downtime deploys
- Canary and blue-green releases
- Automated rollbacks
