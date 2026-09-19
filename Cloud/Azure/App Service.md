---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# App Service

## Up
- [[Azure]]

**Azure App Service** — Fully managed platform (PaaS) to build, deploy and scale web apps, APIs and mobile backends.

**Category:** Compute

## Key concepts
- App Service Plans (tiers)
- Deployment slots
- Custom domains and managed TLS
- Autoscale
- Built-in auth (Easy Auth)
- Continuous deployment

## Common CLI
| Command | Description |
|---|---|
| `az webapp up` | Build and deploy an app |
| `az webapp create` | Create a web app |
| `az webapp deployment slot create` | Create a slot |
| `az webapp list` | List web apps |

## Pricing model
Per App Service Plan instance-hour by tier.

## Works well with
- Azure DNS, Key Vault
- Application Insights
- GitHub / Azure DevOps

## Use cases
- Web apps and APIs
- Managed hosting
- CI/CD web deployments
