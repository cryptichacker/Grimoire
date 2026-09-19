---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Lighthouse

## Up
- [[Azure]]

**Azure Lighthouse** — Enables cross-tenant management so service providers and enterprises can manage many customers' resources at scale.

**Category:** Management

## Key concepts
- Delegated resource management
- Cross-tenant RBAC
- Managed services offers (Marketplace/ARM)
- Least-privilege delegation
- Auditing across tenants

## Common CLI
| Command | Description |
|---|---|
| `az managedservices definition create` | Create a registration definition |
| `az managedservices assignment create` | Assign delegated access |
| `az managedservices assignment list` | List assignments |

## Pricing model
Free.

## Works well with
- Azure RBAC
- Azure Policy
- Resource Graph

## Use cases
- Managed service providers
- Multi-tenant management
- Centralized operations
