---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Applications

## Up
- [[Azure]]

**Azure Managed Applications** — Lets you offer cloud solutions that are easy for customers to deploy and operate, with the publisher managing the resources.

**Category:** Management

## Key concepts
- Managed application definitions
- Service catalog vs Marketplace
- Managed resource group
- Publisher access and control
- Billing integration

## Common CLI
| Command | Description |
|---|---|
| `az managedapp definition create` | Create a definition |
| `az managedapp create` | Deploy a managed application |
| `az managedapp list` | List managed apps |

## Pricing model
Free service; pay for the deployed resources (plus publisher fees).

## Works well with
- Azure Resource Manager / Bicep
- Service Catalog
- Marketplace

## Use cases
- Packaged solutions
- ISV/managed offerings
- Turnkey deployments
