---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure DevTest Labs

## Up
- [[Azure]]

**Azure DevTest Labs** — Quickly creates managed dev/test environments with cost controls and reusable templates.

**Category:** Developer Tools

## Key concepts
- Labs and VMs
- Formulas and custom images
- Policies and quotas (cost control)
- Auto-shutdown schedules
- Artifacts and environments

## Common CLI
| Command | Description |
|---|---|
| `az lab create` | Create a lab (extension) |
| `az lab vm create` | Create a lab VM |
| `az lab vm list` | List lab VMs |

## Pricing model
Free service; pay for the VMs/resources in the lab.

## Works well with
- Virtual Machines
- Azure DevOps
- Resource Manager templates

## Use cases
- Dev/test environments
- Cost-controlled sandboxes
- Training/demo labs
