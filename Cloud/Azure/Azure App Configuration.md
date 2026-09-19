---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure App Configuration

## Up
- [[Azure]]

**Azure App Configuration** — A managed service to centrally manage application settings and feature flags separate from code.

**Category:** Developer Tools

## Key concepts
- Key-value settings
- Feature flags/management
- Labels for environments
- Point-in-time snapshots
- Key Vault references
- Configuration refresh

## Common CLI
| Command | Description |
|---|---|
| `az appconfig create` | Create a store |
| `az appconfig kv set` | Set a key-value |
| `az appconfig kv list` | List settings |
| `az appconfig feature set` | Manage a feature flag |

## Pricing model
Per store per day by tier plus requests.

## Works well with
- App Service, Functions, AKS
- Key Vault
- Azure DevOps / GitHub

## Use cases
- Centralized config
- Feature flags
- Environment-specific settings
