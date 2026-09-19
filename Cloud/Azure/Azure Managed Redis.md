---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Redis

## Up
- [[Azure]]

**Azure Managed Redis** — A next-generation, fully managed Redis service (built on Redis Enterprise) offering higher performance and features.

**Category:** Database

## Key concepts
- Redis Enterprise tiers
- Active geo-replication
- Redis modules (Search, JSON, TimeSeries)
- Clustering
- High availability
- Vector search

## Common CLI
| Command | Description |
|---|---|
| `az redisenterprise create` | Create a Managed Redis cluster |
| `az redisenterprise database create` | Create a database |
| `az redisenterprise list` | List clusters |

## Pricing model
Per cluster capacity/hour by tier.

## Works well with
- App Service / AKS
- Azure SQL / Cosmos DB
- Azure AI (vector search)

## Use cases
- High-performance caching
- Vector/AI data
- Enterprise Redis features
