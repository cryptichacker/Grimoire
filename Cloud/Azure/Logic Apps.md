---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Logic Apps

## Up
- [[Azure]]

**Azure Logic Apps** — A low-code/no-code platform to automate workflows and integrate apps, data and services with connectors.

**Category:** Integration

## Key concepts
- Workflows and triggers/actions
- 400+ managed connectors
- Consumption vs Standard hosting
- Stateful and stateless workflows
- B2B/EDI (Integration Account)
- Designer and code view

## Common CLI
| Command | Description |
|---|---|
| `az logic workflow create` | Create a workflow |
| `az logic workflow list` | List workflows |
| `az logic workflow show` | Show a workflow |

## Pricing model
Per action/connector execution (Consumption) or hosting plan (Standard).

## Works well with
- Service Bus, Event Grid
- Functions
- Microsoft 365 / SaaS connectors

## Use cases
- Workflow automation
- App/SaaS integration
- B2B/EDI processing
