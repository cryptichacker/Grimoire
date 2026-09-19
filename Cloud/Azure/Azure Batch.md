---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Batch

## Up
- [[Azure]]

**Azure Batch** — Runs large-scale parallel and high-performance computing (HPC) batch jobs across pools of managed VMs.

**Category:** Compute

## Key concepts
- Pools, jobs and tasks
- Auto-scaling formulas
- Low-priority/Spot nodes
- Application packages
- Task dependencies

## Common CLI
| Command | Description |
|---|---|
| `az batch account create` | Create a Batch account |
| `az batch pool create` | Create a compute pool |
| `az batch job create` | Create a job |
| `az batch task create` | Add a task |

## Pricing model
Free service; pay for the underlying VM compute and storage.

## Works well with
- Storage
- Virtual Machines / Scale Sets
- Container Registry

## Use cases
- HPC and simulations
- Rendering/transcoding
- Large parallel processing
