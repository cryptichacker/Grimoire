---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Resource Graph

## Up
- [[Azure]]

**Azure Resource Graph** — Fast, at-scale querying of Azure resources across subscriptions using a Kusto-like query language.

**Category:** Management

## Key concepts
- Resource explorer queries (KQL)
- Cross-subscription/tenant scope
- Change history
- Shared queries
- Powering Azure portal search

## Common CLI
| Command | Description |
|---|---|
| `az graph query -q "Resources | project name, type"` | Run a resource query |
| `az graph shared-query create` | Save a shared query |

## Pricing model
Free.

## Works well with
- Azure Policy
- Cost Management
- Management Groups

## Use cases
- Inventory at scale
- Governance reporting
- Resource discovery
