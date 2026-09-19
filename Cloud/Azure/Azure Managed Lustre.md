---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Lustre

## Up
- [[Azure]]

**Azure Managed Lustre** — A managed, fully featured Lustre parallel file system for high-performance computing workloads.

**Category:** Storage

## Key concepts
- Managed Lustre file systems
- High throughput and IOPS
- Blob integration (import/export)
- Short-lived or persistent clusters
- HPC-optimized

## Common CLI
| Command | Description |
|---|---|
| `az amlfs create` | Create a Managed Lustre file system |
| `az amlfs list` | List file systems |
| `az amlfs show` | Show details |

## Pricing model
Per provisioned TiB by throughput tier.

## Works well with
- Blob Storage
- CycleCloud / Batch
- AKS / VMs

## Use cases
- HPC scratch storage
- AI/ML training data
- Big-compute file systems
