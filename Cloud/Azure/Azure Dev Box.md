---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Dev Box

## Up
- [[Azure]]

**Microsoft Dev Box** — Provides secure, ready-to-code cloud developer workstations preconfigured for project-specific tasks.

**Category:** Developer Tools

## Key concepts
- Dev centers and projects
- Dev box definitions (images/SKUs)
- Network connections
- Self-service provisioning
- Managed via Intune
- Auto-stop schedules

## Common CLI
| Command | Description |
|---|---|
| `az devcenter admin devbox-definition create` | Define a dev box |
| `az devcenter dev dev-box create` | Provision a dev box |
| `az devcenter dev dev-box list` | List dev boxes |

## Pricing model
Per dev box compute/storage-hour.

## Works well with
- Deployment Environments
- Entra ID / Intune
- Azure DevOps / GitHub

## Use cases
- Cloud dev workstations
- Onboarding developers
- Consistent dev environments
