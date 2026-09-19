---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Private Link

## Up
- [[Azure]]

**Azure Private Link** — Provides private connectivity from a VNet to Azure PaaS, customer or partner services over the Microsoft backbone.

**Category:** Networking

## Key concepts
- Private endpoints (private IP in your VNet)
- Private Link services (behind Standard LB)
- Private DNS integration
- No public exposure
- Cross-tenant access

## Common CLI
| Command | Description |
|---|---|
| `az network private-endpoint create` | Create a private endpoint |
| `az network private-link-service create` | Publish a private link service |
| `az network private-endpoint list` | List private endpoints |

## Pricing model
Per endpoint-hour plus per-GB data processed.

## Works well with
- Virtual Network
- Storage, SQL, Key Vault
- Load Balancer

## Use cases
- Private access to PaaS
- Exposing your own services privately
- Avoiding internet exposure
