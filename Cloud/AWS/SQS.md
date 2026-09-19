---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# SQS

## Up
- [[AWS]]

**Amazon Simple Queue Service** — Fully managed message queues that decouple producers and consumers and buffer work reliably.

**Category:** Application Integration

## Key concepts
- Standard vs FIFO queues
- Visibility timeout
- Long polling
- Dead-letter queues
- Message retention and delay queues
- Batch send/receive

## Common CLI
| Command | Description |
|---|---|
| `aws sqs create-queue` | Create a queue |
| `aws sqs send-message` | Enqueue a message |
| `aws sqs receive-message` | Dequeue messages |
| `aws sqs delete-message` | Acknowledge/delete a message |

## Pricing model
Per million requests (FIFO slightly higher).

## Works well with
- Lambda (event source)
- SNS
- EventBridge, Auto Scaling

## Use cases
- Work queues
- Buffering traffic spikes
- Decoupling microservices
