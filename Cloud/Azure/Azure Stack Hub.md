---
tags: [cloud, azure, hybrid]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Stack Hub

## Up
- [[Azure]]

**Azure Stack Hub** — An integrated system that runs a consistent subset of Azure services in your own datacenter (connected or disconnected).

**Category:** Hybrid

## Key concepts
- On-prem Azure services (IaaS/PaaS subset)
- Consistent Azure APIs/portal
- Connected or disconnected modes
- Marketplace syndication
- Operator and tenant roles

## Common CLI
| Command | Description |
|---|---|
| `az cloud register` | Register the Azure Stack Hub cloud endpoint |
| `az vm create` | Deploy resources with the same CLI |
| `Admin portal` | Operate the stamp |

## Pricing model
Pay-as-you-use (connected) or capacity-based licensing.

## Works well with
- Azure (consistent tooling)
- Azure Arc
- Virtual Machines / App Service

## Use cases
- Disconnected/edge Azure
- Data-sovereignty workloads
- Consistent hybrid apps
