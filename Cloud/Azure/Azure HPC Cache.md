---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure HPC Cache

## Up
- [[Azure]]

**Azure HPC Cache** — A file-caching service that speeds access to data for high-performance computing from Blob or on-prem NAS.

**Category:** Storage

## Key concepts
- Cache with configurable throughput
- Aggregated namespace
- Blob and NFS storage targets
- Hot data acceleration
- Read-heavy HPC workloads

## Common CLI
| Command | Description |
|---|---|
| `az hpc-cache create` | Create an HPC Cache |
| `az hpc-cache list` | List caches |
| `az hpc-cache blob-storage-target add` | Add a storage target |

## Pricing model
Per cache throughput tier per hour.

## Works well with
- Blob Storage / NAS
- CycleCloud / Batch
- VMs / AKS

## Use cases
- HPC data acceleration
- Bursting to cloud compute
- Read-heavy pipelines
