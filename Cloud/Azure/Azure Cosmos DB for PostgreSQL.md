---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Cosmos DB for PostgreSQL

## Up
- [[Azure]]

**Azure Cosmos DB for PostgreSQL** — Distributed PostgreSQL powered by the Citus extension for horizontal scale-out of relational workloads.

**Category:** Database

## Key concepts
- Distributed tables (sharding via Citus)
- Coordinator and worker nodes
- Single-node to multi-node scale
- PostgreSQL compatibility
- High availability

## Common CLI
| Command | Description |
|---|---|
| `az cosmosdb postgres cluster create` | Create a cluster |
| `az cosmosdb postgres cluster list` | List clusters |
| `az cosmosdb postgres cluster show` | Show cluster details |

## Pricing model
Per node (coordinator/worker) compute and storage.

## Works well with
- PostgreSQL clients
- App Service / AKS
- Power BI

## Use cases
- Scale-out PostgreSQL
- Multi-tenant SaaS
- Real-time operational analytics
