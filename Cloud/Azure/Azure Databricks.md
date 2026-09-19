---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Databricks

## Up
- [[Azure]]

**Azure Databricks** — A first-party Apache Spark-based analytics and lakehouse platform optimized for Azure.

**Category:** Analytics

## Key concepts
- Workspaces and clusters
- Notebooks and jobs
- Delta Lake and Unity Catalog
- MLflow for ML lifecycle
- SQL warehouses
- Auto-scaling and Photon engine

## Common CLI
| Command | Description |
|---|---|
| `az databricks workspace create` | Create a workspace |
| `az databricks workspace list` | List workspaces |
| `databricks jobs create` | Create a job (Databricks CLI) |

## Pricing model
Per Databricks Unit (DBU) plus underlying VM compute.

## Works well with
- Data Lake Storage
- Data Factory
- Power BI, Machine Learning

## Use cases
- Lakehouse analytics
- Big-data ETL
- Machine learning at scale
