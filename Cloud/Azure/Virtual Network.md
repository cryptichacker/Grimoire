---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Virtual Network

## Up
- [[Azure]]

**Azure Virtual Network (VNet)** — Your isolated private network in Azure where resources securely communicate, with subnets and routing.

**Category:** Networking

## Key concepts
- Address space and subnets
- Network Security Groups (NSGs)
- Route tables (UDRs)
- VNet peering
- Service and private endpoints
- DDoS and Network Watcher

## Common CLI
| Command | Description |
|---|---|
| `az network vnet create` | Create a VNet |
| `az network vnet subnet create` | Create a subnet |
| `az network vnet peering create` | Peer two VNets |
| `az network nsg create` | Create an NSG |

## Pricing model
The VNet is free; NAT Gateway, peering data and gateways cost.

## Works well with
- Virtual Machines
- Load Balancer
- Private Link, VPN Gateway

## Use cases
- Network isolation
- Multi-tier architectures
- Hybrid networking
