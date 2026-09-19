---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Transit Gateway

## Up
- [[AWS]]

**AWS Transit Gateway** — A cloud router that hub-and-spoke connects thousands of VPCs and on-premises networks through a single gateway.

**Category:** Networking

## Key concepts
- Attachments (VPC/VPN/DX/peering)
- Route tables per gateway
- Inter-region peering
- Multicast support
- Centralized egress and inspection
- Sharing across accounts via RAM

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-transit-gateway` | Create a transit gateway |
| `aws ec2 create-transit-gateway-vpc-attachment` | Attach a VPC |
| `aws ec2 create-transit-gateway-route` | Add a route |

## Pricing model
Per attachment-hour plus per GB of data processed.

## Works well with
- VPC
- Site-to-Site VPN, Direct Connect
- Resource Access Manager

## Use cases
- Large multi-VPC networks
- Centralized connectivity
- Network segmentation
