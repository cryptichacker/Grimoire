---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Developer CLI

## Up
- [[Azure]]

**Azure Developer CLI (azd)** — A developer-centric command-line tool that accelerates going from local code to a running app on Azure.

**Category:** Developer Tools

## Key concepts
- Templates (azd init)
- Infrastructure as code (Bicep/Terraform)
- Provision + deploy in one flow
- Environments
- CI/CD pipeline config
- Local run and monitor

## Common CLI
| Command | Description |
|---|---|
| `azd init` | Initialize from a template |
| `azd up` | Provision and deploy |
| `azd deploy` | Deploy app code |
| `azd pipeline config` | Set up CI/CD |

## Pricing model
Free (pay for the provisioned resources).

## Works well with
- Bicep / Terraform
- App Service / Container Apps / AKS
- GitHub Actions / Azure DevOps

## Use cases
- Fast app bootstrapping
- Repeatable environments
- Local-to-cloud workflow
