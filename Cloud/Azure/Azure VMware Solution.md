---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure VMware Solution

## Up
- [[Azure]]

**Azure VMware Solution (AVS)** — Runs native VMware environments (vSphere, vSAN, NSX) on dedicated Azure infrastructure for lift-and-shift.

**Category:** Compute

## Key concepts
- Private clouds and clusters
- Native VMware stack (vCenter/NSX-T)
- Stretched clusters
- HCX migration
- Connectivity to Azure services

## Common CLI
| Command | Description |
|---|---|
| `az vmware private-cloud create` | Create a private cloud |
| `az vmware private-cloud list` | List private clouds |
| `az vmware cluster create` | Add a cluster |

## Pricing model
Per dedicated host-hour (reserved terms lower cost).

## Works well with
- ExpressRoute
- Virtual Network
- Azure Backup / Site Recovery

## Use cases
- VMware migration
- Data-center extension
- Hybrid VMware ops
