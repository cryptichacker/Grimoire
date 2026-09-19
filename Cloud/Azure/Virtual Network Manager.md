---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Virtual Network Manager

## Up
- [[Azure]]

**Azure Virtual Network Manager** — Centrally manages connectivity and security-rule configurations for virtual networks across subscriptions and Regions.

**Category:** Networking

## Key concepts
- Network groups (static/dynamic)
- Connectivity configurations (mesh/hub-spoke)
- Security admin rules
- Scopes and management groups
- Deployment across Regions

## Common CLI
| Command | Description |
|---|---|
| `az network manager create` | Create a network manager |
| `az network manager group create` | Create a network group |
| `az network manager connectivity-configuration create` | Create connectivity config |

## Pricing model
Per managed subscription/VNet (charges apply).

## Works well with
- Virtual Network
- Azure Firewall
- Management Groups

## Use cases
- At-scale network config
- Hub-spoke automation
- Central security rules
