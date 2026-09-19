---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Gateway Load Balancer

## Up
- [[Azure]]

**Azure Gateway Load Balancer** — A load-balancer SKU that transparently inserts third-party network virtual appliances into the traffic path.

**Category:** Networking

## Key concepts
- Gateway SKU load balancer
- Service chaining of NVAs
- Transparent (bump-in-the-wire) insertion
- VXLAN tunneling
- High availability for appliances

## Common CLI
| Command | Description |
|---|---|
| `az network lb create --sku Gateway` | Create a Gateway Load Balancer |
| `az network lb address-pool create` | Create the appliance pool |
| `az network nic ip-config update --gateway-lb` | Chain traffic through it |

## Pricing model
Per LB-hour plus per-GB processed.

## Works well with
- Network virtual appliances
- Standard Load Balancer
- Virtual Network

## Use cases
- Inline security appliances
- Transparent NVA insertion
- Firewall/IDS chaining
