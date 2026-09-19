---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Template Specs

## Up
- [[Azure]]

**Azure Template Specs** — Stores ARM/Bicep templates as first-class Azure resources for sharing, versioning and controlled deployment.

**Category:** Management

## Key concepts
- Template spec resources and versions
- RBAC-controlled sharing
- Deploy directly from a spec
- Integration with Bicep
- No external storage needed

## Common CLI
| Command | Description |
|---|---|
| `az ts create` | Create/update a template spec |
| `az ts show` | Show a template spec |
| `az deployment group create --template-spec` | Deploy from a spec |

## Pricing model
Free (pay for the deployed resources).

## Works well with
- Azure Resource Manager / Bicep
- Managed Applications
- Azure DevOps / GitHub

## Use cases
- Sharing IaC templates
- Versioned deployments
- Governed template reuse
