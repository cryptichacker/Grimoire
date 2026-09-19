---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Service Bus

## Up
- [[Azure]]

**Azure Service Bus** — Enterprise message broker with queues and publish/subscribe topics for reliable, ordered messaging.

**Category:** Integration

## Key concepts
- Queues and topics/subscriptions
- Sessions (FIFO) and dead-lettering
- Message deferral and scheduling
- Transactions and duplicate detection
- Tiers (Basic/Standard/Premium)
- Geo-disaster recovery

## Common CLI
| Command | Description |
|---|---|
| `az servicebus namespace create` | Create a namespace |
| `az servicebus queue create` | Create a queue |
| `az servicebus topic create` | Create a topic |
| `az servicebus topic subscription create` | Create a subscription |

## Pricing model
Per operation (Standard) or messaging-unit-hour (Premium).

## Works well with
- Functions, Logic Apps
- App Service / AKS
- Event Grid

## Use cases
- Enterprise messaging
- Order-preserving queues
- Decoupling services
