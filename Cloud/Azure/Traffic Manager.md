---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Traffic Manager

## Up
- [[Azure]]

**Azure Traffic Manager** — DNS-based traffic load balancer that distributes traffic across global endpoints for availability and performance.

**Category:** Networking

## Key concepts
- Profiles and endpoints
- Routing methods (priority/weighted/performance/geographic)
- Endpoint health monitoring
- DNS TTL control
- Nested profiles

## Common CLI
| Command | Description |
|---|---|
| `az network traffic-manager profile create` | Create a profile |
| `az network traffic-manager endpoint create` | Add an endpoint |
| `az network traffic-manager profile show` | Show a profile |

## Pricing model
Per million DNS queries plus health checks.

## Works well with
- App Service, VMs
- Front Door
- Public endpoints

## Use cases
- Global DNS routing
- Multi-region failover
- Latency-based routing
