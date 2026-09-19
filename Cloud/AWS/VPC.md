---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# VPC

## Up
- [[AWS]]

**Amazon Virtual Private Cloud** — Your own isolated virtual network in AWS where you define IP ranges, subnets, routing and gateways.

**Category:** Networking

## Key concepts
- CIDR blocks and subnets (public/private)
- Route tables
- Internet Gateway and NAT Gateway
- Security groups (stateful) vs NACLs (stateless)
- VPC peering
- Endpoints (Gateway/Interface/PrivateLink) and Flow Logs

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-vpc` | Create a VPC |
| `aws ec2 create-subnet` | Add a subnet |
| `aws ec2 create-internet-gateway` | Create an internet gateway |
| `aws ec2 create-route` | Add a route |

## Pricing model
The VPC is free; NAT gateways, interface endpoints and data transfer incur charges.

## Works well with
- EC2, Elastic Load Balancing
- RDS
- Transit Gateway, Direct Connect

## Use cases
- Network isolation
- Multi-tier architectures
- Hybrid networking
