---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# VPC Lattice

## Up
- [[AWS]]

**Amazon VPC Lattice** — Application networking service that connects, secures and monitors service-to-service communication across VPCs and accounts.

**Category:** Networking

## Key concepts
- Service networks
- Services and target groups
- IAM auth policies
- Listeners and routing rules
- Cross-account sharing via RAM
- Layer-7 routing

## Common CLI
| Command | Description |
|---|---|
| `aws vpc-lattice create-service-network` | Create a service network |
| `aws vpc-lattice create-service` | Create a service |
| `aws vpc-lattice create-target-group` | Create a target group |

## Pricing model
Per service-network/service-hour plus data processed and requests.

## Works well with
- VPC
- IAM
- EKS, Elastic Load Balancing

## Use cases
- Microservice connectivity
- Cross-account service mesh
- Simplified east-west networking
