---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EventBridge

## Up
- [[AWS]]

**Amazon EventBridge** — Serverless event bus that routes events between AWS services, SaaS applications and your own apps.

**Category:** Application Integration

## Key concepts
- Event buses (default, custom, partner)
- Rules and event patterns
- Targets
- Schema registry
- Pipes (source-to-target with filtering/enrichment)
- Scheduler (cron/rate)

## Common CLI
| Command | Description |
|---|---|
| `aws events put-rule` | Create a rule |
| `aws events put-targets` | Attach targets to a rule |
| `aws events put-events` | Publish custom events |

## Pricing model
Per million events for custom/partner events; many AWS-service events are free.

## Works well with
- Lambda, Step Functions
- SQS/SNS
- SaaS partner sources

## Use cases
- Event-driven architectures
- Scheduling
- SaaS event ingestion
