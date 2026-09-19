---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Arc

## Up
- [[Azure]]

**Azure Arc** — Extends Azure management and governance to servers, Kubernetes clusters and data services anywhere — on-prem or multicloud.

**Category:** Management

## Key concepts
- Arc-enabled servers
- Arc-enabled Kubernetes
- Arc-enabled data services
- Policy/RBAC/Monitor everywhere
- GitOps configuration
- Extended Security Updates

## Common CLI
| Command | Description |
|---|---|
| `az connectedmachine list` | List Arc-enabled servers |
| `az connectedk8s connect` | Onboard a Kubernetes cluster |
| `az connectedk8s list` | List connected clusters |

## Pricing model
Free for basic control-plane; per-resource charges for some Arc-enabled services.

## Works well with
- Azure Policy
- Azure Monitor
- Defender for Cloud

## Use cases
- Hybrid/multicloud management
- Consistent governance
- On-prem Kubernetes/servers in Azure
