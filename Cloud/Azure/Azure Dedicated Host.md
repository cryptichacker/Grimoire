---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Dedicated Host

## Up
- [[Azure]]

**Azure Dedicated Host** — Provides physical servers dedicated to a single customer for hosting Azure VMs, aiding compliance and licensing.

**Category:** Compute

## Key concepts
- Host groups and dedicated hosts
- Single-tenant physical isolation
- Control over maintenance windows
- Bring-your-own-license (Windows/SQL)
- Availability zones

## Common CLI
| Command | Description |
|---|---|
| `az vm host group create` | Create a host group |
| `az vm host create` | Create a dedicated host |
| `az vm create --host` | Place a VM on a host |

## Pricing model
Per dedicated host-hour (VMs incur no extra compute charge).

## Works well with
- Virtual Machines
- Azure Hybrid Benefit
- Availability Zones

## Use cases
- Regulatory isolation
- BYOL scenarios
- Physical-server control
