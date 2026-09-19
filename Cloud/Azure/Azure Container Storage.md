---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Container Storage

## Up
- [[Azure]]

**Azure Container Storage** — A managed, container-native storage service for provisioning and managing persistent volumes for AKS at scale.

**Category:** Storage

## Key concepts
- Storage pools
- Backing options (Elastic SAN, local NVMe, Azure Disks)
- Persistent volumes for Kubernetes
- Dynamic provisioning
- Snapshots

## Common CLI
| Command | Description |
|---|---|
| `az k8s-extension create --extension-type microsoft.azurecontainerstorage` | Install on AKS |
| `kubectl apply -f storagepool.yaml` | Create a storage pool |

## Pricing model
Per capacity used (plus the backing storage service).

## Works well with
- AKS
- Elastic SAN / Managed Disks
- Local NVMe

## Use cases
- Kubernetes persistent volumes
- Stateful containers
- High-performance container storage
