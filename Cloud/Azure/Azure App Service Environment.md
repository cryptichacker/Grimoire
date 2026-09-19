---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure App Service Environment

## Up
- [[Azure]]

**Azure App Service Environment (ASE)** — A fully isolated, dedicated deployment of Azure App Service into your virtual network for high-scale, secure apps.

**Category:** Compute

## Key concepts
- Single-tenant isolation
- VNet integration (inbound/outbound control)
- Dedicated compute (Isolated v2 plans)
- Large scale-out
- Internal or external load balancing

## Common CLI
| Command | Description |
|---|---|
| `az appservice ase create` | Create an App Service Environment |
| `az appservice ase list` | List ASEs |
| `az appservice plan create --app-service-environment` | Create an Isolated plan in an ASE |

## Pricing model
Per Isolated v2 App Service Plan instance-hour.

## Works well with
- App Service
- Virtual Network
- Application Gateway

## Use cases
- Isolated/regulated web apps
- VNet-only apps
- High-scale hosting
