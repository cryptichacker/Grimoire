---
tags: [cloud, azure, migration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Site Recovery

## Up
- [[Azure]]

**Azure Site Recovery** — Disaster recovery as a service that replicates workloads to a secondary location and orchestrates failover.

**Category:** Migration

## Key concepts
- Recovery Services vault
- Continuous replication
- Recovery plans and orchestration
- Test failover (non-disruptive drills)
- Azure-to-Azure and on-prem-to-Azure

## Common CLI
| Command | Description |
|---|---|
| `az backup vault create` | Create a Recovery Services vault |
| `Portal: Site Recovery replication` | Configure replication and recovery plans |

## Pricing model
Per protected instance per month plus target storage/compute during failover.

## Works well with
- Virtual Machines
- Azure Backup
- VMware/Hyper-V (on-prem)

## Use cases
- Disaster recovery
- Business continuity
- Datacenter failover
