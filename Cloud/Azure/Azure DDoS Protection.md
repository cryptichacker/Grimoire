---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure DDoS Protection

## Up
- [[Azure]]

**Azure DDoS Protection** — Protects Azure resources from distributed denial-of-service attacks with always-on monitoring and mitigation.

**Category:** Identity & Security

## Key concepts
- DDoS Network Protection vs IP Protection
- Adaptive tuning
- Attack metrics and alerts
- Mitigation policies
- Cost protection
- Rapid Response support

## Common CLI
| Command | Description |
|---|---|
| `az network ddos-protection create` | Create a protection plan |
| `az network ddos-protection list` | List plans |
| `az network vnet update --ddos-protection-plan` | Associate with a VNet |

## Pricing model
Per protection plan per month (covers a set of resources) or per protected IP.

## Works well with
- Virtual Network
- Public IPs
- Application Gateway / Front Door

## Use cases
- DDoS mitigation
- Protecting public endpoints
- Attack cost protection
