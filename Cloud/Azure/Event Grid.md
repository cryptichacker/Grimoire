---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Event Grid

## Up
- [[Azure]]

**Azure Event Grid** — Fully managed event routing service that delivers events from sources to handlers with pub/sub at scale.

**Category:** Integration

## Key concepts
- System and custom topics
- Event subscriptions and filters
- Event schemas (CloudEvents)
- Push delivery with retries and dead-lettering
- Domains
- MQTT broker (namespaces)

## Common CLI
| Command | Description |
|---|---|
| `az eventgrid topic create` | Create a custom topic |
| `az eventgrid event-subscription create` | Create a subscription |
| `az eventgrid topic list` | List topics |

## Pricing model
Per operation (per million events).

## Works well with
- Functions, Logic Apps
- Service Bus / Event Hubs
- Storage, Container Apps

## Use cases
- Event-driven architectures
- Reacting to Azure events
- Serverless workflows
