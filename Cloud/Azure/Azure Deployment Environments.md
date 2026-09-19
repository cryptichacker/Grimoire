---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Deployment Environments

## Up
- [[Azure]]

**Azure Deployment Environments** — Lets teams quickly spin up app infrastructure environments from curated, template-based catalogs.

**Category:** Developer Tools

## Key concepts
- Dev centers and projects
- Environment definitions (IaC catalogs)
- Environment types (dev/test/prod)
- Self-service for developers
- Governance and RBAC

## Common CLI
| Command | Description |
|---|---|
| `az devcenter admin environment-type create` | Create an environment type |
| `az devcenter dev environment create` | Create an environment |
| `az devcenter dev environment list` | List environments |

## Pricing model
Free service; pay for the deployed resources.

## Works well with
- Dev Box
- Azure Resource Manager / Bicep
- Azure DevOps / GitHub

## Use cases
- Self-service environments
- Standardized infra templates
- Platform engineering
