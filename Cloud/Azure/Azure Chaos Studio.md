---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Chaos Studio

## Up
- [[Azure]]

**Azure Chaos Studio** — A managed chaos-engineering service to inject faults and measure application resilience through controlled experiments.

**Category:** Management

## Key concepts
- Experiments and steps
- Faults (service-direct and agent-based)
- Targets and selectors
- Blast-radius control
- Integration with monitoring for hypotheses

## Common CLI
| Command | Description |
|---|---|
| `Portal: Chaos Studio experiment` | Create and run experiments |
| `az rest / ARM` | Manage experiments via API |

## Pricing model
Per fault-action-minute during experiments.

## Works well with
- Azure Monitor
- Virtual Machines / AKS
- Load Testing

## Use cases
- Chaos engineering
- Resilience validation
- Failover testing
