---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Synapse Analytics

## Up
- [[Azure]]

**Azure Synapse Analytics** — An integrated analytics platform combining data warehousing, big-data (Spark) and data integration.

**Category:** Analytics

## Key concepts
- Dedicated vs serverless SQL pools
- Apache Spark pools
- Synapse Pipelines (data integration)
- Synapse Studio
- Synapse Link (to Cosmos DB/SQL)
- Data Lake integration

## Common CLI
| Command | Description |
|---|---|
| `az synapse workspace create` | Create a workspace |
| `az synapse sql pool create` | Create a dedicated SQL pool |
| `az synapse spark pool create` | Create a Spark pool |

## Pricing model
Per dedicated SQL pool DWU-hour, serverless per TB scanned, and Spark vCore-hours.

## Works well with
- Data Lake Storage
- Power BI
- Cosmos DB (Synapse Link)

## Use cases
- Data warehousing
- Big-data analytics
- Unified analytics platform
