---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Cosmos DB for MongoDB

## Up
- [[Azure]]

**Azure Cosmos DB for MongoDB** — MongoDB-compatible database available as the request-unit API or a vCore-based cluster for familiar MongoDB workloads.

**Category:** Database

## Key concepts
- RU-based vs vCore-based (dedicated cluster)
- MongoDB wire-protocol compatibility
- Sharding and replica sets (vCore)
- Vector search
- Automatic backups
- Global distribution (RU)

## Common CLI
| Command | Description |
|---|---|
| `az cosmosdb mongocluster create` | Create a vCore cluster |
| `az cosmosdb create --kind MongoDB` | Create an RU-based account |
| `az cosmosdb mongocluster list` | List clusters |

## Pricing model
Per RU/s or per vCore-cluster compute plus storage.

## Works well with
- MongoDB clients
- App Service / AKS
- Azure AI (vector search)

## Use cases
- MongoDB workloads
- App migrations from MongoDB
- Vector/AI apps
