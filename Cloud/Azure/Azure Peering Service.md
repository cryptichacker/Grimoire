---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Peering Service

## Up
- [[Azure]]

**Azure Peering Service** — Enhances connectivity to Microsoft cloud services (Microsoft 365, Azure PaaS) over the public internet via partner ISPs.

**Category:** Networking

## Key concepts
- Optimized routing via peering partners
- Latency and route monitoring
- Registered prefixes
- Redundant connectivity
- Not a private circuit (public internet)

## Common CLI
| Command | Description |
|---|---|
| `az peering service create` | Create a peering service |
| `az peering service prefix create` | Register a prefix |
| `az peering service list` | List peering services |

## Pricing model
Per registered prefix/connection.

## Works well with
- Microsoft 365 / Azure PaaS
- ExpressRoute (alternative)
- Network Watcher

## Use cases
- Optimized SaaS connectivity
- Reliable internet routing
- Enterprise branch access
