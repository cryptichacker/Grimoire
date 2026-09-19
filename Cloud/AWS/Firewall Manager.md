---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Firewall Manager

## Up
- [[AWS]]

**AWS Firewall Manager** — Centrally configures and manages firewall rules (WAF, Shield, Network Firewall, security groups) across an organization.

**Category:** Security

## Key concepts
- Security policies
- Automatic application to new resources
- Compliance reporting
- Requires Organizations and Config
- Delegated administration

## Common CLI
| Command | Description |
|---|---|
| `aws fms put-policy` | Create or update a policy |
| `aws fms list-policies` | List policies |
| `aws fms get-compliance-detail` | Check compliance |

## Pricing model
Per policy per Region per month.

## Works well with
- WAF, Shield Advanced
- Network Firewall
- Organizations, Config

## Use cases
- Org-wide firewall governance
- Consistent WAF rules
- Compliance enforcement
