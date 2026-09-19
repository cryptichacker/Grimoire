---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure DNS Private Resolver

## Up
- [[Azure]]

**Azure DNS Private Resolver** — Enables querying Azure private DNS zones and resolving names between on-prem and Azure without custom DNS servers.

**Category:** Networking

## Key concepts
- Inbound and outbound endpoints
- DNS forwarding rulesets
- Conditional forwarding
- VNet linking
- No DNS-server VMs to manage

## Common CLI
| Command | Description |
|---|---|
| `az dns-resolver create` | Create a private resolver |
| `az dns-resolver inbound-endpoint create` | Create an inbound endpoint |
| `az dns-resolver outbound-endpoint create` | Create an outbound endpoint |

## Pricing model
Per resolver endpoint-hour plus queries processed.

## Works well with
- Private DNS zones
- Virtual Network
- VPN / ExpressRoute

## Use cases
- Hybrid DNS resolution
- Private zone queries from on-prem
- Replacing DNS-server VMs
