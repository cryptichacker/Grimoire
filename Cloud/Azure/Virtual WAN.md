---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Virtual WAN

## Up
- [[Azure]]

**Azure Virtual WAN** — A networking service that brings connectivity, security and routing into a single, managed hub-and-spoke fabric.

**Category:** Networking

## Key concepts
- Virtual hubs
- Hub-and-spoke connectivity
- Branch (VPN/SD-WAN) and ExpressRoute connections
- Secured hubs (Azure Firewall)
- Any-to-any routing

## Common CLI
| Command | Description |
|---|---|
| `az network vwan create` | Create a Virtual WAN |
| `az network vhub create` | Create a virtual hub |
| `az network vhub connection create` | Connect a VNet |

## Pricing model
Per hub-hour plus connection units and data processed.

## Works well with
- Virtual Network, VPN/ExpressRoute Gateway
- Azure Firewall
- Virtual Network peering

## Use cases
- Global transit networking
- Branch connectivity
- Secured hub networking
