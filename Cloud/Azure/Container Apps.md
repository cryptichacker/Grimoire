---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Container Apps

## Up
- [[Azure]]

**Azure Container Apps** — Serverless container platform for microservices and event-driven apps built on Kubernetes and KEDA, without managing it.

**Category:** Compute

## Key concepts
- Environments
- Revisions and traffic splitting
- KEDA autoscaling (scale to zero)
- Dapr integration
- Ingress
- Jobs

## Common CLI
| Command | Description |
|---|---|
| `az containerapp create` | Create an app |
| `az containerapp update` | Update an app |
| `az containerapp revision list` | List revisions |
| `az containerapp env create` | Create an environment |

## Pricing model
Per vCPU-second and GB-second of active/idle usage plus requests.

## Works well with
- Azure Container Registry
- Dapr, Event Grid
- Key Vault

## Use cases
- Microservices
- Event-driven apps
- Background jobs
