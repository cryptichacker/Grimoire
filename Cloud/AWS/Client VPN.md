---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Client VPN

## Up
- [[AWS]]

**AWS Client VPN** — Managed, elastic OpenVPN-based remote-access VPN that lets users securely reach AWS and on-prem resources.

**Category:** Networking

## Key concepts
- Client VPN endpoints
- Authentication (AD, SAML, mutual certificate)
- Authorization rules
- Split vs full tunnel
- Connection logging

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-client-vpn-endpoint` | Create an endpoint |
| `aws ec2 associate-client-vpn-target-network` | Associate a subnet |
| `aws ec2 authorize-client-vpn-ingress` | Add an authorization rule |

## Pricing model
Per endpoint-association-hour plus per active client-connection-hour.

## Works well with
- VPC
- Directory Service / IAM Identity Center
- Transit Gateway

## Use cases
- Remote workforce access
- Secure developer access
- Replacing self-managed VPNs
