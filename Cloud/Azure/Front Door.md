---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Front Door

## Up
- [[Azure]]

**Azure Front Door** — Global, scalable entry point that uses Microsoft's edge network for fast, secure web delivery and load balancing.

**Category:** Networking

## Key concepts
- Global HTTP/HTTPS load balancing
- CDN caching at the edge
- WAF integration
- Routing rules and origin groups
- TLS/SSL offload
- Health probes and failover

## Common CLI
| Command | Description |
|---|---|
| `az afd profile create` | Create a Front Door profile |
| `az afd endpoint create` | Create an endpoint |
| `az afd origin-group create` | Create an origin group |
| `az afd route create` | Create a route |

## Pricing model
Per GB delivered and per request plus routing rules/WAF.

## Works well with
- App Service, Storage (static sites)
- Application Gateway
- WAF

## Use cases
- Global web delivery
- CDN and acceleration
- Global load balancing
