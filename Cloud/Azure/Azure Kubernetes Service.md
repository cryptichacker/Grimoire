---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Kubernetes Service

## Up
- [[Azure]]

**Azure Kubernetes Service (AKS)** — Managed Kubernetes with an Azure-operated control plane and managed node pools.

**Category:** Compute

## Key concepts
- System and user node pools
- Managed control plane
- Cluster autoscaler and KEDA
- Workload identity (Entra)
- Azure CNI vs kubenet
- Add-ons (monitoring, ingress)

## Common CLI
| Command | Description |
|---|---|
| `az aks create` | Create a cluster |
| `az aks get-credentials` | Configure kubectl |
| `az aks scale` | Scale a node pool |
| `az aks nodepool add` | Add a node pool |

## Pricing model
Free control plane (or paid SLA tier); pay for node VMs.

## Works well with
- Azure Container Registry
- Microsoft Entra ID
- Azure Monitor, Load Balancer

## Use cases
- Container orchestration
- Microservices
- Portable workloads
