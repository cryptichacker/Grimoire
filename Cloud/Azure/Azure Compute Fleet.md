---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Compute Fleet

## Up
- [[Azure]]

**Azure Compute Fleet** — Provisions and manages large, mixed fleets of Spot and on-demand VMs across sizes and zones for capacity at scale.

**Category:** Compute

## Key concepts
- Mixed instance sizes and families
- Spot and on-demand mix
- Allocation strategies
- Capacity across zones
- Large-scale provisioning

## Common CLI
| Command | Description |
|---|---|
| `az compute-fleet create` | Create a compute fleet |
| `az compute-fleet list` | List fleets |
| `az compute-fleet show` | Show fleet details |

## Pricing model
No extra charge; pay for the underlying VM capacity (Spot discounts apply).

## Works well with
- Virtual Machine Scale Sets
- Spot VMs
- Batch / AKS

## Use cases
- Large batch/compute fleets
- Spot-optimized capacity
- Diversified VM provisioning
