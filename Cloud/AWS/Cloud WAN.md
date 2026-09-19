---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cloud WAN

## Up
- [[AWS]]

**AWS Cloud WAN** — Builds, manages and monitors a unified global network across VPCs, Regions and on-prem from a central dashboard.

**Category:** Networking

## Key concepts
- Global network and core network
- Network segments (isolation)
- Policy-driven configuration
- Attachments (VPC/VPN/TGW/Connect)
- Central monitoring

## Common CLI
| Command | Description |
|---|---|
| `aws networkmanager create-global-network` | Create a global network |
| `aws networkmanager create-core-network` | Create a core network |
| `aws networkmanager create-site` | Register a site |

## Pricing model
Per core-network-edge and attachment plus data processed.

## Works well with
- Transit Gateway
- VPC
- Direct Connect, Site-to-Site VPN

## Use cases
- Global WAN
- Multi-Region networking
- Segmented enterprise networks
