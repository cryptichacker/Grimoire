---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Red Hat OpenShift

## Up
- [[Azure]]

**Azure Red Hat OpenShift (ARO)** — A jointly engineered, fully managed Red Hat OpenShift (Kubernetes) service on Azure.

**Category:** Compute

## Key concepts
- Managed OpenShift clusters
- Operators and OperatorHub
- Built-in CI/CD (OpenShift Pipelines)
- Integrated registry
- Jointly supported by Microsoft and Red Hat

## Common CLI
| Command | Description |
|---|---|
| `az aro create` | Create an ARO cluster |
| `az aro list` | List clusters |
| `az aro show` | Show cluster details |

## Pricing model
Per cluster-hour plus worker node VMs and OpenShift licensing.

## Works well with
- Container Registry
- Entra ID
- Azure Monitor

## Use cases
- OpenShift on Azure
- Enterprise Kubernetes
- Hybrid container platforms
