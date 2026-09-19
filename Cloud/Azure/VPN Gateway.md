---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# VPN Gateway

## Up
- [[Azure]]

**Azure VPN Gateway** — Sends encrypted traffic between an Azure VNet and on-premises or between VNets over the internet.

**Category:** Networking

## Key concepts
- Site-to-site and VNet-to-VNet
- Point-to-site (user VPN)
- Active-active gateways
- BGP routing
- Gateway SKUs
- IPsec/IKE tunnels

## Common CLI
| Command | Description |
|---|---|
| `az network vnet-gateway create` | Create a VPN gateway |
| `az network vpn-connection create` | Create a connection |
| `az network local-gateway create` | Define the on-prem gateway |

## Pricing model
Per gateway-hour by SKU plus data transfer.

## Works well with
- Virtual Network
- ExpressRoute (coexistence)
- VWAN

## Use cases
- Hybrid connectivity
- Remote user VPN
- Cross-region VNet links
