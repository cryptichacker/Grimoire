---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Bicep

## Up
- [[Azure]]

**Azure Bicep** — A domain-specific language for declaratively deploying Azure resources, transpiling to ARM templates.

**Category:** Developer Tools

## Key concepts
- Modules and parameters
- Resource declarations and references
- Transpiles to ARM JSON
- What-if deployments
- Bicep registry
- Deployment stacks

## Common CLI
| Command | Description |
|---|---|
| `az bicep build` | Compile Bicep to ARM |
| `az deployment group create --template-file main.bicep` | Deploy Bicep |
| `az deployment group what-if` | Preview changes |

## Pricing model
Free (pay for the deployed resources).

## Works well with
- Azure Resource Manager
- Deployment Environments
- Azure DevOps / GitHub Actions

## Use cases
- Infrastructure as code
- Repeatable deployments
- Modular Azure infra
