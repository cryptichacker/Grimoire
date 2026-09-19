---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Web Application Firewall

## Up
- [[Azure]]

**Azure Web Application Firewall (WAF)** — Centralized protection of web apps from common exploits and vulnerabilities, deployed on Application Gateway or Front Door.

**Category:** Identity & Security

## Key concepts
- WAF policies
- Managed rule sets (OWASP CRS)
- Custom rules and rate limiting
- Bot protection
- Detection vs prevention mode
- Geo-match and IP restrictions

## Common CLI
| Command | Description |
|---|---|
| `az network application-gateway waf-policy create` | Create a WAF policy (App Gateway) |
| `az network front-door waf-policy create` | Create a WAF policy (Front Door) |
| `az network application-gateway waf-policy managed-rule rule-set add` | Add a managed rule set |

## Pricing model
Per policy and per rule plus request/traffic charges on the host service.

## Works well with
- Application Gateway
- Front Door
- Firewall Manager

## Use cases
- Block SQLi/XSS
- Rate limiting and bot control
- App-layer protection
