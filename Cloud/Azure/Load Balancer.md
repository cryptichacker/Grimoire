---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Load Balancer

## Up
- [[Azure]]

**Azure Load Balancer** — High-performance, low-latency Layer-4 (TCP/UDP) load balancing for inbound and outbound traffic.

**Category:** Networking

## Key concepts
- Public vs internal
- Frontend IPs and backend pools
- Health probes
- Load-balancing and outbound rules
- Standard vs Basic SKU
- Zone redundancy

## Common CLI
| Command | Description |
|---|---|
| `az network lb create` | Create a load balancer |
| `az network lb rule create` | Add a rule |
| `az network lb probe create` | Add a health probe |
| `az network lb list` | List balancers |

## Pricing model
Per rule and per GB processed (Standard SKU).

## Works well with
- VMs / Scale Sets
- Virtual Network
- Public IP addresses

## Use cases
- L4 load balancing
- High availability
- Outbound connectivity
