---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Direct Connect

## Up
- [[AWS]]

**AWS Direct Connect** — Dedicated, private network connection between your data center and AWS for consistent bandwidth and latency.

**Category:** Networking

## Key concepts
- Dedicated vs hosted connections
- Virtual interfaces: private, public, transit
- Link aggregation groups (LAG)
- Direct Connect Gateway
- Encryption via VPN over DX; MACsec

## Common CLI
| Command | Description |
|---|---|
| `aws directconnect describe-connections` | List connections |
| `aws directconnect create-private-virtual-interface` | Create a private VIF |

## Pricing model
Per port-hour by capacity plus data transfer out (cheaper than internet egress).

## Works well with
- VPC
- Transit Gateway
- Site-to-Site VPN

## Use cases
- Hybrid cloud
- Large or consistent data transfer
- Low-latency private links
