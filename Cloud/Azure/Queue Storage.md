---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Queue Storage

## Up
- [[Azure]]

**Azure Queue Storage** — Simple, durable message queue storage for decoupling application components at scale.

**Category:** Storage

## Key concepts
- Queues and messages
- At-least-once delivery
- Visibility timeout
- 64 KB message size
- Poison-message handling
- Part of a storage account

## Common CLI
| Command | Description |
|---|---|
| `az storage queue create` | Create a queue |
| `az storage message put` | Enqueue a message |
| `az storage message get` | Dequeue a message |
| `az storage queue list` | List queues |

## Pricing model
Per operation and storage used.

## Works well with
- Functions (trigger)
- VMs / App Service
- Logic Apps

## Use cases
- Work queues
- Buffering spikes
- Simple async messaging
