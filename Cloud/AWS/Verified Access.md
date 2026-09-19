---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Verified Access

## Up
- [[AWS]]

**AWS Verified Access** — Provides secure, VPN-less access to corporate applications based on identity and device posture (zero trust).

**Category:** Security

## Key concepts
- Verified Access instances
- Trust providers (identity/device)
- Access policies (Cedar)
- Per-application endpoints
- Detailed access logging

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-verified-access-instance` | Create an instance |
| `aws ec2 create-verified-access-group` | Create a policy group |
| `aws ec2 create-verified-access-endpoint` | Expose an application |

## Pricing model
Per application-hour plus per GB processed.

## Works well with
- IAM Identity Center / IdPs
- Device-trust providers
- Elastic Load Balancing

## Use cases
- Zero-trust app access
- VPN replacement
- Contextual access control
