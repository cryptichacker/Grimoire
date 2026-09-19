---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Resource Mover

## Up
- [[Azure]]

**Azure Resource Mover** — Simplifies moving resources across Azure Regions with dependency awareness and validation.

**Category:** Management

## Key concepts
- Move collections
- Cross-Region resource moves
- Dependency detection
- Prepare/initiate/commit workflow
- Test before commit

## Common CLI
| Command | Description |
|---|---|
| `az resource-mover move-collection create` | Create a move collection |
| `az resource-mover move-resource add` | Add a resource to move |
| `az resource-mover move-collection prepare` | Prepare the move |

## Pricing model
Free service; pay for any resources created during the move.

## Works well with
- Virtual Machines / networking
- Site Recovery
- Resource Graph

## Use cases
- Cross-Region moves
- Region consolidation
- DR/relocation planning
