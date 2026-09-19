---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# ParallelCluster

## Up
- [[AWS]]

**AWS ParallelCluster** — Open-source cluster-management tool that provisions and manages HPC clusters on AWS from a simple config file.

**Category:** Compute

## Key concepts
- Cluster config (YAML)
- Head node and compute fleets
- Schedulers (Slurm)
- Elastic scaling of compute nodes
- Shared storage (FSx for Lustre/EFS)

## Common CLI
| Command | Description |
|---|---|
| `pcluster create-cluster` | Create an HPC cluster |
| `pcluster list-clusters` | List clusters |
| `pcluster delete-cluster` | Delete a cluster |

## Pricing model
Free tool; pay for the EC2, storage and networking the cluster uses.

## Works well with
- EC2 (Spot)
- FSx for Lustre
- Batch

## Use cases
- HPC and simulations
- Research computing
- Tightly-coupled workloads
