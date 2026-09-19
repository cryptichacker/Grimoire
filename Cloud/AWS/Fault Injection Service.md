---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Fault Injection Service

## Up
- [[AWS]]

**AWS Fault Injection Service** — A managed chaos-engineering service to run fault-injection experiments and improve application resilience.

**Category:** Management

## Key concepts
- Experiment templates
- Actions (stop instances, add latency, etc.)
- Targets and target selection
- Stop conditions (CloudWatch alarms)
- Safety controls

## Common CLI
| Command | Description |
|---|---|
| `aws fis create-experiment-template` | Define an experiment |
| `aws fis start-experiment` | Run an experiment |
| `aws fis list-experiments` | List experiments |

## Pricing model
Per action-minute during experiments.

## Works well with
- EC2, ECS/EKS, RDS
- CloudWatch (stop conditions)
- Resilience Hub

## Use cases
- Chaos engineering
- Resilience testing
- Validating failover
