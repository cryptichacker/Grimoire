---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Elastic Load Balancing

## Up
- [[AWS]]

**Elastic Load Balancing (ELB)** — Automatically distributes incoming traffic across healthy targets in multiple AZs, with three balancer types.

**Category:** Networking

## Key concepts
- Application LB (L7 HTTP/HTTPS)
- Network LB (L4 TCP/UDP, static IP)
- Gateway LB (for appliances)
- Target groups and health checks
- Listeners and routing rules
- Cross-zone balancing, sticky sessions, TLS termination

## Common CLI
| Command | Description |
|---|---|
| `aws elbv2 create-load-balancer` | Create an ALB or NLB |
| `aws elbv2 create-target-group` | Create a target group |
| `aws elbv2 create-listener` | Add a listener |

## Pricing model
Per load-balancer-hour plus Load Balancer Capacity Units consumed.

## Works well with
- EC2, ECS/EKS
- ACM, WAF
- Auto Scaling

## Use cases
- Highly available web tiers
- Microservice routing
- TCP/UDP load balancing
