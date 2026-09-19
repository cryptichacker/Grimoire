---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Modeling and Simulation Workbench

## Up
- [[Azure]]

**Azure Modeling and Simulation Workbench** — A managed, secure environment for engineering modeling and simulation (MODSIM) collaboration and HPC workloads.

**Category:** Compute

## Key concepts
- Workbenches and chambers (secure enclaves)
- Managed HPC/desktop compute
- Data pipelines with controlled egress
- Collaboration with IP protection
- Preinstalled EDA/CAE tooling support

## Common CLI
| Command | Description |
|---|---|
| `Portal/ARM: create a workbench` | Provision the environment |
| `az resource create` | Deploy via ARM template |

## Pricing model
Per compute/storage consumed by the workbench.

## Works well with
- HPC (CycleCloud/Batch)
- NetApp Files / Managed Lustre
- Virtual Network

## Use cases
- Engineering simulation
- Secure IP collaboration
- EDA/CAE workloads
