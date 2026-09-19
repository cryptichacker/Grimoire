---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Entra Private Access

## Up
- [[Azure]]

**Microsoft Entra Private Access** — A Zero Trust Network Access (ZTNA) service (part of Global Secure Access) for secure access to private apps without a VPN.

**Category:** Identity & Security

## Key concepts
- ZTNA per-app access
- Connectors to private networks
- Conditional Access per app
- No full-network VPN
- Quick Access and app segments

## Common CLI
| Command | Description |
|---|---|
| `Portal: Global Secure Access` | Publish private apps |
| `Connector install` | Deploy private network connectors |

## Pricing model
Per user per month (Entra Suite / SSE licensing).

## Works well with
- Microsoft Entra ID
- Conditional Access
- Entra Internet Access

## Use cases
- VPN replacement (ZTNA)
- Private app access
- Zero Trust networking
