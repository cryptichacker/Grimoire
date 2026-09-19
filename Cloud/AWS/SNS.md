---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# SNS

## Up
- [[AWS]]

**Amazon Simple Notification Service** — Publish/subscribe messaging that pushes messages to many subscribers such as queues, functions, HTTP endpoints, email and SMS.

**Category:** Application Integration

## Key concepts
- Topics (standard and FIFO)
- Subscriptions and protocols
- Message filtering by attributes
- Fan-out to multiple SQS queues
- Dead-letter queues
- Mobile push notifications

## Common CLI
| Command | Description |
|---|---|
| `aws sns create-topic` | Create a topic |
| `aws sns subscribe` | Add a subscriber |
| `aws sns publish` | Send a message |

## Pricing model
Per million requests plus delivery charges (SMS and email extra).

## Works well with
- SQS
- Lambda
- EventBridge, CloudWatch alarms

## Use cases
- Fan-out notifications
- Operational alerts
- Decoupled messaging
