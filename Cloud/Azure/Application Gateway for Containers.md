---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Application Gateway for Containers

## Up
- [[Azure]]

**Application Gateway for Containers** — A next-generation application (L7) load balancer and ingress controller optimized for Kubernetes/AKS workloads.

**Category:** Networking

## Key concepts
- Application Load Balancer (ALB) resource
- Gateway API and Ingress support
- Traffic splitting and header routing
- Managed by the ALB controller
- Per-pod backend targeting

## Common CLI
| Command | Description |
|---|---|
| `az network alb create` | Create an Application Load Balancer |
| `az network alb frontend create` | Create a frontend |
| `az network alb association create` | Associate a subnet |

## Pricing model
Per ALB-hour plus capacity units and requests.

## Works well with
- AKS
- Virtual Network
- Web Application Firewall

## Use cases
- Kubernetes ingress
- Advanced L7 routing
- Blue-green/canary for containers
