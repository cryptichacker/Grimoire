---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Step Functions

## Up
- [[AWS]]

**AWS Step Functions** — Coordinates multiple AWS services into serverless workflows defined as state machines.

**Category:** Application Integration

## Key concepts
- State machine defined in Amazon States Language
- States: Task, Choice, Parallel, Map, Wait
- Standard vs Express workflows
- Built-in error handling and retries
- Direct service integrations
- Callback (wait-for-token) patterns

## Common CLI
| Command | Description |
|---|---|
| `aws stepfunctions create-state-machine` | Create a workflow |
| `aws stepfunctions start-execution` | Start an execution |
| `aws stepfunctions describe-execution` | Check execution status |

## Pricing model
Standard: per state transition; Express: per request plus duration.

## Works well with
- Lambda, ECS
- SNS/SQS, DynamoDB
- Glue, EventBridge

## Use cases
- Service orchestration
- ETL pipelines
- Long-running processes
