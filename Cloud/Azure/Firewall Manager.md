---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Firewall Manager

## Up
- [[Azure]]

**Azure Firewall Manager** — Centrally manages security policies and route management for Azure Firewall across hubs and VNets.

**Category:** Identity & Security

## Key concepts
- Firewall policies (hierarchical)
- Secured virtual hubs
- Centralized route management
- Third-party SECaaS integration
- DDoS and WAF policy association

## Common CLI
| Command | Description |
|---|---|
| `az network firewall policy create` | Create a firewall policy |
| `az network firewall policy rule-collection-group create` | Manage rule collections |
| `az network firewall policy show` | Show a policy |

## Pricing model
Per policy and per secured hub (plus underlying Azure Firewall).

## Works well with
- Azure Firewall
- Virtual WAN
- Web Application Firewall

## Use cases
- Central firewall governance
- Multi-hub security
- Consistent network policy
