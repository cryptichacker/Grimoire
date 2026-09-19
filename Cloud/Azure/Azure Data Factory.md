---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Data Factory

## Up
- [[Azure]]

**Azure Data Factory** — A cloud data-integration service to orchestrate and automate data movement and transformation (ETL/ELT).

**Category:** Analytics

## Key concepts
- Pipelines and activities
- 90+ connectors
- Mapping and wrangling data flows
- Integration runtimes (Azure/self-hosted/SSIS)
- Triggers and scheduling
- CI/CD integration

## Common CLI
| Command | Description |
|---|---|
| `az datafactory create` | Create a data factory |
| `az datafactory pipeline create` | Create a pipeline |
| `az datafactory pipeline create-run` | Run a pipeline |

## Pricing model
Per pipeline activity run, data-flow vCore-hour and data movement.

## Works well with
- Data Lake Storage, Synapse
- Databricks
- SQL / Cosmos DB

## Use cases
- ETL/ELT pipelines
- Data migration
- Hybrid data integration
