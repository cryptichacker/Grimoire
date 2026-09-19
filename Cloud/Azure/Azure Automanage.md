---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Automanage

## Up
- [[Azure]]

**Azure Automanage** — Automatically applies and maintains best-practice configurations (backup, monitoring, security) for VMs.

**Category:** Management

## Key concepts
- Configuration profiles
- Best-practice baselines
- Machine configuration (guest config)
- Drift detection and remediation
- Hotpatch and update integration

## Common CLI
| Command | Description |
|---|---|
| `az automanage configuration-profile create` | Create a profile |
| `az automanage configuration-profile-assignment create` | Assign to a VM |
| `az automanage configuration-profile list` | List profiles |

## Pricing model
Free for VM best practices; some included services may cost.

## Works well with
- Virtual Machines
- Azure Backup / Monitor
- Azure Policy

## Use cases
- VM best practices automation
- Configuration compliance
- Reduced ops toil
