---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Resource Manager

## Up
- [[Azure]]

**Azure Resource Manager (ARM)** — The deployment and management layer for Azure — the control plane through which all resources are created and governed.

**Category:** Management

## Key concepts
- Resource groups and subscriptions
- ARM/Bicep templates (IaC)
- Role-based access control (RBAC)
- Tags and locks
- Management groups
- Deployment stacks

## Common CLI
| Command | Description |
|---|---|
| `az group create` | Create a resource group |
| `az deployment group create` | Deploy a template/Bicep |
| `az resource list` | List resources |
| `az lock create` | Lock a resource |

## Pricing model
Free (pay for the resources deployed).

## Works well with
- Bicep / Terraform
- Azure Policy
- Blueprints / Landing Zones

## Use cases
- Infrastructure as code
- Resource organization
- Access control and tagging
