---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Elastic SAN

## Up
- [[Azure]]

**Azure Elastic SAN** — A fully managed, cloud-native storage area network (SAN) offering scalable, high-throughput block storage over iSCSI.

**Category:** Storage

## Key concepts
- Elastic SAN, volume groups and volumes
- iSCSI connectivity
- Scalable IOPS/throughput
- Private endpoints
- Snapshots

## Common CLI
| Command | Description |
|---|---|
| `az elastic-san create` | Create an Elastic SAN |
| `az elastic-san volume-group create` | Create a volume group |
| `az elastic-san volume create` | Create a volume |

## Pricing model
Per provisioned capacity and performance (base + additional).

## Works well with
- Virtual Machines / AKS
- Virtual Network
- Container Storage

## Use cases
- Large-scale block storage
- Databases and I/O-heavy apps
- SAN migration
