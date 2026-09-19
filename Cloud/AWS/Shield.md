---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Shield

## Up
- [[AWS]]

**AWS Shield** — Managed DDoS protection: Standard is automatic and free, while Advanced adds higher-layer defenses and support.

**Category:** Security

## Key concepts
- Shield Standard (automatic L3/L4)
- Shield Advanced (subscription)
- DDoS Response Team access
- Cost protection for scaling during attacks
- Health-based detection
- WAF integration

## Common CLI
| Command | Description |
|---|---|
| `aws shield subscribe` | Enable Shield Advanced for the account |
| `aws shield list-protections` | List protected resources |
| `aws shield create-protection` | Protect a resource |

## Pricing model
Standard is free; Advanced is a monthly subscription plus data fees.

## Works well with
- CloudFront, Route 53
- Elastic Load Balancing
- Global Accelerator, WAF

## Use cases
- DDoS mitigation
- Protecting public endpoints
- Attack cost protection
