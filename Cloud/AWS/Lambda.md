---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Lambda

## Up
- [[AWS]]

**AWS Lambda** — Run functions in response to events with no servers to manage; AWS handles scaling, patching and availability, and you pay only while code runs.

**Category:** Compute

## Key concepts
- Handler function invoked per event
- Event sources / triggers (sync and async)
- Cold starts vs warm invocations
- Concurrency, reserved and provisioned concurrency
- Layers for shared dependencies
- Execution role for permissions
- 15-minute max timeout; memory sizing also sets CPU

## Common CLI
| Command | Description |
|---|---|
| `aws lambda create-function` | Create a function |
| `aws lambda invoke` | Invoke synchronously |
| `aws lambda update-function-code` | Deploy new code |
| `aws lambda list-functions` | List functions |

## Pricing model
Per request plus GB-seconds of compute; generous always-free tier.

## Works well with
- API Gateway, EventBridge
- S3, DynamoDB Streams
- SQS, SNS

## Use cases
- Serverless APIs and backends
- Event/stream processing
- Automation and scheduled jobs
