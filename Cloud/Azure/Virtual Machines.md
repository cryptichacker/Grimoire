---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Virtual Machines

## Up
- [[Azure]]

**Azure Virtual Machines** — On-demand, scalable Linux and Windows virtual machines with full control over the OS and configuration.

**Category:** Compute

## Key concepts
- VM sizes/series (B, D, E, F, N-GPU)
- Images (Marketplace/custom/gallery)
- OS and data Managed Disks
- Availability sets and zones
- NSGs and network interfaces
- Spot and Reserved Instances

## Common CLI
| Command | Description |
|---|---|
| `az vm create` | Create a VM |
| `az vm list` | List VMs |
| `az vm start / stop / deallocate` | Power operations |
| `az vm resize` | Change VM size |

## Pricing model
Per-second/hour by size and OS; cheaper via Spot, Reserved Instances or Savings Plans.

## Works well with
- Managed Disks
- Virtual Network, Load Balancer
- VM Scale Sets

## Use cases
- Servers and applications
- Lift-and-shift
- Dev/test environments
