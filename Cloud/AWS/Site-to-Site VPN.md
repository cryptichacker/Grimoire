---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Site-to-Site VPN

## Up
- [[AWS]]

**AWS Site-to-Site VPN** — Creates encrypted IPsec tunnels between your on-premises network and a VPC over the internet.

**Category:** Networking

## Key concepts
- Customer gateway and virtual private gateway (or Transit Gateway)
- Two tunnels for redundancy
- Static vs BGP routing
- IKE/IPsec
- Accelerated VPN option

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-customer-gateway` | Define your on-prem side |
| `aws ec2 create-vpn-gateway` | Create a virtual private gateway |
| `aws ec2 create-vpn-connection` | Create the VPN connection |

## Pricing model
Per VPN connection-hour plus data transfer.

## Works well with
- VPC
- Transit Gateway
- Direct Connect

## Use cases
- Hybrid connectivity
- Backup path for Direct Connect
- Branch-office links
