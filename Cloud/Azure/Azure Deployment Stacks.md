---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Deployment Stacks

## Up
- [[Azure]]

**Azure Deployment Stacks** — Manages a collection of resources as a single deployable and governable unit with lifecycle and deny controls.

**Category:** Management

## Key concepts
- Stack as a managed unit
- Deny settings (protect managed resources)
- Update and delete cascade
- Scope: resource group/subscription/management group
- Bicep/ARM based

## Common CLI
| Command | Description |
|---|---|
| `az stack group create` | Create/update a resource-group stack |
| `az stack sub create` | Create a subscription-scope stack |
| `az stack group delete` | Delete a stack and its resources |

## Pricing model
Free (pay for the deployed resources).

## Works well with
- Azure Resource Manager / Bicep
- Azure Policy
- Template Specs

## Use cases
- Lifecycle of resource groups
- Preventing drift/deletion
- Governed IaC deployments
