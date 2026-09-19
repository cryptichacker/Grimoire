---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# VPC IPAM

## Up
- [[AWS]]

**Amazon VPC IP Address Manager** — Plans, tracks and monitors IP address usage across your AWS environment from a central console.

**Category:** Networking

## Key concepts
- IPAM pools and scopes
- Automatic allocation to VPCs
- Utilization monitoring and alarms
- Cross-account/Region via Organizations
- Historical IP records

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-ipam` | Create an IPAM |
| `aws ec2 create-ipam-pool` | Create an address pool |
| `aws ec2 allocate-ipam-pool-cidr` | Allocate a CIDR |

## Pricing model
Per active IP address monitored (advanced tier).

## Works well with
- VPC
- Organizations
- Transit Gateway

## Use cases
- IP address planning
- Avoiding CIDR conflicts
- Multi-account IP governance
