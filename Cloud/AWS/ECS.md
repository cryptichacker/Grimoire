---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# ECS

## Up
- [[AWS]]

**Amazon Elastic Container Service** — Run and scale Docker containers on AWS using task definitions and services, on EC2 you manage or on serverless Fargate.

**Category:** Compute

## Key concepts
- Task definition (container spec)
- Task (running unit) vs Service (desired count + scheduler)
- Cluster grouping capacity
- Launch type: EC2 or Fargate
- Task role for per-task permissions
- Service discovery and ALB integration

## Common CLI
| Command | Description |
|---|---|
| `aws ecs create-cluster` | Create a cluster |
| `aws ecs register-task-definition` | Define a task |
| `aws ecs create-service` | Run and maintain tasks |
| `aws ecs update-service` | Scale or deploy |
| `aws ecs list-tasks` | List running tasks |

## Pricing model
No charge for the control plane; pay for the underlying EC2 or Fargate resources.

## Works well with
- Fargate, ECR
- Elastic Load Balancing
- CloudWatch, IAM

## Use cases
- Microservices
- Batch and scheduled jobs
- Long-running services
