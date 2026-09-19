---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Fargate

## Up
- [[AWS]]

**AWS Fargate** — Serverless compute for ECS and EKS: specify CPU/memory per task and AWS runs the containers with no nodes to provision or patch.

**Category:** Compute

## Key concepts
- Task-level CPU/memory sizing
- No node/cluster capacity management
- Per-second billing
- Strong per-task isolation
- Fargate Spot for cheaper capacity
- awsvpc networking (ENI per task)

## Common CLI
| Command | Description |
|---|---|
| `aws ecs run-task --launch-type FARGATE` | Run a one-off task on Fargate |
| `aws ecs create-service --launch-type FARGATE` | Run a service on Fargate |

## Pricing model
Per vCPU-second and GB-second consumed by the task; Fargate Spot is discounted.

## Works well with
- ECS, EKS
- ECR, Elastic Load Balancing
- CloudWatch

## Use cases
- Bursty or irregular workloads
- Teams avoiding node operations
- Isolated per-task workloads
