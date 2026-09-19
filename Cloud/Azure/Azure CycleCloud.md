---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure CycleCloud

## Up
- [[Azure]]

**Azure CycleCloud** — An orchestration and management tool for deploying and operating HPC and big-compute clusters on Azure.

**Category:** Compute

## Key concepts
- Cluster templates
- Scheduler support (Slurm, PBS, LSF, Grid Engine)
- Autoscaling based on job queue
- Cost controls
- Deployed as a management application

## Common CLI
| Command | Description |
|---|---|
| `cyclecloud start_cluster` | Start a cluster (CycleCloud CLI) |
| `cyclecloud show_cluster` | Show cluster status |
| `(Marketplace)` | Deploy the CycleCloud app |

## Pricing model
Free tool; pay for the underlying VM compute and storage.

## Works well with
- Virtual Machine Scale Sets
- Azure Batch
- NetApp Files / Managed Lustre

## Use cases
- HPC cluster management
- Scheduler-based workloads
- Elastic big-compute
