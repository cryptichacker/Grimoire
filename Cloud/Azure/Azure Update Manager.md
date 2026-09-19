---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Update Manager

## Up
- [[Azure]]

**Azure Update Manager** — A unified service to assess and deploy operating-system updates across Azure, on-prem and multicloud machines.

**Category:** Management

## Key concepts
- Assessment and one-time updates
- Scheduled patching (maintenance configurations)
- Hotpatching support
- Arc-enabled server coverage
- Compliance reporting

## Common CLI
| Command | Description |
|---|---|
| `az vm assess-patches` | Assess pending updates on a VM |
| `az maintenance configuration create` | Create a patch schedule |
| `az vm install-patches` | Install updates now |

## Pricing model
Free for Azure VMs; per-server/hour for Arc-enabled machines.

## Works well with
- Virtual Machines
- Azure Arc
- Azure Policy

## Use cases
- Patch management
- Update compliance
- Scheduled maintenance
