---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Functions

## Up
- [[Azure]]

**Azure Functions** — Event-driven serverless compute that runs code on triggers without managing infrastructure.

**Category:** Compute

## Key concepts
- Triggers and bindings
- Consumption / Premium / Dedicated plans
- Durable Functions (orchestration)
- Cold starts
- Multiple language stacks

## Common CLI
| Command | Description |
|---|---|
| `az functionapp create` | Create a function app |
| `func start` | Run locally (Core Tools) |
| `az functionapp deployment source config-zip` | Deploy code |
| `az functionapp list` | List function apps |

## Pricing model
Per execution and GB-seconds (Consumption); Premium/Dedicated by instance.

## Works well with
- Event Grid, Service Bus
- Storage, Cosmos DB
- Application Insights

## Use cases
- Event processing
- APIs and webhooks
- Automation and glue code
