---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Route Server

## Up
- [[Azure]]

**Azure Route Server** — Simplifies dynamic routing between your network virtual appliances (NVAs) and your virtual network using BGP.

**Category:** Networking

## Key concepts
- BGP peering with NVAs
- Automatic route exchange
- No manual UDR management
- Branch-to-branch routing
- Integration with ExpressRoute/VPN gateways

## Common CLI
| Command | Description |
|---|---|
| `az network routeserver create` | Create a Route Server |
| `az network routeserver peering create` | Create a BGP peering |
| `az network routeserver list` | List Route Servers |

## Pricing model
Per Route Server-hour plus data processed.

## Works well with
- Virtual Network
- Network Virtual Appliances
- VPN / ExpressRoute Gateway

## Use cases
- Dynamic routing with NVAs
- Simplified route management
- SD-WAN integration
