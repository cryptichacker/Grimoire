---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Cosmos DB

## Up
- [[Azure]]

**Azure Cosmos DB** — Globally distributed, multi-model NoSQL (and relational) database with turnkey global replication and low latency.

**Category:** Database

## Key concepts
- APIs: NoSQL, MongoDB, Cassandra, Gremlin, Table
- Partition keys and throughput (RU/s)
- Provisioned vs serverless vs autoscale
- Multi-region writes
- 5 consistency levels
- Change feed

## Common CLI
| Command | Description |
|---|---|
| `az cosmosdb create` | Create an account |
| `az cosmosdb sql database create` | Create a database |
| `az cosmosdb sql container create` | Create a container |
| `az cosmosdb list` | List accounts |

## Pricing model
Per provisioned RU/s (or per request serverless) plus storage.

## Works well with
- Functions (change feed)
- AKS / App Service
- Synapse Link

## Use cases
- Global low-latency apps
- High-scale NoSQL
- Event sourcing
