---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Instance for Apache Cassandra

## Up
- [[Azure]]

**Azure Managed Instance for Apache Cassandra** — A managed service for running and scaling open-source Apache Cassandra clusters, including hybrid deployments.

**Category:** Database

## Key concepts
- Clusters and datacenters
- Managed operations and patching
- Hybrid clusters with on-prem
- Autoscaling and turnkey scaling
- Backups

## Common CLI
| Command | Description |
|---|---|
| `az managed-cassandra cluster create` | Create a cluster |
| `az managed-cassandra datacenter create` | Add a datacenter |
| `az managed-cassandra cluster list` | List clusters |

## Pricing model
Per node-hour plus storage.

## Works well with
- Virtual Network
- Cosmos DB (Cassandra API)
- Azure Monitor

## Use cases
- Cassandra workloads
- Hybrid Cassandra
- High-scale wide-column data
