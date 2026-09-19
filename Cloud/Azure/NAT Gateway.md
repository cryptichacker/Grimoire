---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# NAT Gateway

## Up
- [[Azure]]

**Azure NAT Gateway** — Provides highly scalable, resilient outbound internet connectivity for resources in a subnet using SNAT.

**Category:** Networking

## Key concepts
- Subnet association
- Public IPs / prefixes
- Scalable SNAT ports
- Zonal deployment
- No inbound connectivity

## Common CLI
| Command | Description |
|---|---|
| `az network nat gateway create` | Create a NAT gateway |
| `az network vnet subnet update --nat-gateway` | Associate with a subnet |
| `az network nat gateway list` | List NAT gateways |

## Pricing model
Per gateway-hour plus per-GB data processed.

## Works well with
- Virtual Network
- Public IP addresses
- VM Scale Sets

## Use cases
- Reliable outbound connectivity
- SNAT port exhaustion fix
- Consistent egress IPs
