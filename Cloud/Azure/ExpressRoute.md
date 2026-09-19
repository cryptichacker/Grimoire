---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# ExpressRoute

## Up
- [[Azure]]

**Azure ExpressRoute** — Extends on-premises networks into Azure over a private, dedicated connection from a connectivity provider.

**Category:** Networking

## Key concepts
- Circuits and peerings (private/Microsoft)
- Bandwidth tiers
- ExpressRoute Global Reach
- FastPath
- Redundant connections
- Provider vs Direct

## Common CLI
| Command | Description |
|---|---|
| `az network express-route create` | Create a circuit |
| `az network express-route peering create` | Configure peering |
| `az network express-route list` | List circuits |

## Pricing model
Per circuit (metered or unlimited data plan) plus provider fees.

## Works well with
- Virtual Network (gateway)
- VPN Gateway (coexistence)
- Microsoft 365 (peering)

## Use cases
- Private hybrid connectivity
- Consistent low latency
- Large data transfer
