---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Container Instances

## Up
- [[Azure]]

**Azure Container Instances** — Run containers on demand in seconds without managing servers or orchestrators.

**Category:** Compute

## Key concepts
- Container groups
- Per-second billing
- Fast startup
- Persistent volumes (Azure Files)
- VNet deployment
- Restart policies

## Common CLI
| Command | Description |
|---|---|
| `az container create` | Run a container |
| `az container show` | Show details |
| `az container logs` | View logs |
| `az container delete` | Delete |

## Pricing model
Per vCPU-second and GB-second.

## Works well with
- Azure Container Registry
- Azure Files
- Logic Apps, AKS virtual nodes

## Use cases
- Burst/batch jobs
- Simple containers
- Elastic build agents
