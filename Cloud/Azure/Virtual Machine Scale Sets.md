---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Virtual Machine Scale Sets

## Up
- [[Azure]]

**Azure Virtual Machine Scale Sets** — Deploy and manage a group of identical, load-balanced VMs that autoscale with demand.

**Category:** Compute

## Key concepts
- Uniform vs Flexible orchestration
- Autoscale rules
- Rolling/automatic upgrades
- Health probes
- Spot mix and zone spanning

## Common CLI
| Command | Description |
|---|---|
| `az vmss create` | Create a scale set |
| `az vmss scale` | Set instance count |
| `az vmss update-instances` | Roll updates |
| `az vmss list` | List scale sets |

## Pricing model
Free orchestration; pay for the underlying VM instances.

## Works well with
- Load Balancer / Application Gateway
- Azure Monitor autoscale
- Managed Disks

## Use cases
- Elastic web tiers
- Big-compute fleets
- Self-healing capacity
