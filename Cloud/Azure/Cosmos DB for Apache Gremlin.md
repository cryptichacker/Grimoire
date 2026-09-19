---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Cosmos DB for Apache Gremlin

## Up
- [[Azure]]

**Azure Cosmos DB for Apache Gremlin** — A graph database API on Cosmos DB using the Apache TinkerPop Gremlin query language.

**Category:** Database

## Key concepts
- Graph model (vertices/edges)
- Gremlin query language
- Global distribution and RU throughput
- Partitioning
- Automatic indexing

## Common CLI
| Command | Description |
|---|---|
| `az cosmosdb create --capabilities EnableGremlin` | Create a Gremlin-enabled account |
| `az cosmosdb gremlin database create` | Create a graph database |
| `az cosmosdb gremlin graph create` | Create a graph |

## Pricing model
Per provisioned RU/s (or serverless) plus storage.

## Works well with
- Gremlin clients
- App Service / AKS
- Cosmos DB tooling

## Use cases
- Graph workloads
- Recommendation/social graphs
- Connected-data apps
