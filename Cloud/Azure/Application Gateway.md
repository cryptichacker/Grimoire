---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Application Gateway

## Up
- [[Azure]]

**Azure Application Gateway** — Layer-7 (HTTP/HTTPS) load balancer with routing, TLS termination and an optional Web Application Firewall.

**Category:** Networking

## Key concepts
- Listeners and routing rules
- URL path/host-based routing
- TLS termination and end-to-end TLS
- Web Application Firewall (WAF)
- Autoscaling v2 SKU
- Health probes

## Common CLI
| Command | Description |
|---|---|
| `az network application-gateway create` | Create a gateway |
| `az network application-gateway rule create` | Add a rule |
| `az network application-gateway waf-policy create` | Create a WAF policy |

## Pricing model
Per gateway-hour plus capacity units (v2) and WAF.

## Works well with
- VMs / Scale Sets, AKS (AGIC)
- Key Vault (certs)
- Front Door

## Use cases
- Web app load balancing
- Path-based routing
- App-layer WAF
