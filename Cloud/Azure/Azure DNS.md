---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure DNS

## Up
- [[Azure]]

**Azure DNS** — Host your DNS domains in Azure with fast, reliable name resolution using Microsoft's global network.

**Category:** Networking

## Key concepts
- Public and private DNS zones
- Record sets (A/AAAA/CNAME/etc.)
- Alias records to Azure resources
- Private DNS for VNets
- Delegation
- RBAC

## Common CLI
| Command | Description |
|---|---|
| `az network dns zone create` | Create a zone |
| `az network dns record-set a add-record` | Add an A record |
| `az network private-dns zone create` | Create a private zone |
| `az network dns zone list` | List zones |

## Pricing model
Per hosted zone/month plus per million queries.

## Works well with
- App Service, Front Door
- Private Link
- Traffic Manager

## Use cases
- DNS hosting
- Private name resolution
- Alias to Azure resources
