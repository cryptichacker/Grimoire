---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Data Lake Storage

## Up
- [[Azure]]

**Azure Data Lake Storage** — Blob Storage with a hierarchical namespace (Gen2) optimized for big-data analytics at scale.

**Category:** Storage

## Key concepts
- Hierarchical namespace
- POSIX ACLs
- Directories and files
- Optimized for analytics engines
- Tiering and lifecycle
- Built on Blob Storage

## Common CLI
| Command | Description |
|---|---|
| `az storage fs create` | Create a filesystem |
| `az storage fs directory create` | Create a directory |
| `az storage fs file upload` | Upload a file |
| `az storage fs list` | List paths |

## Pricing model
Per GB-month by tier plus transactions (Blob pricing).

## Works well with
- Synapse, Databricks
- HDInsight
- Data Factory

## Use cases
- Data lakes
- Big-data analytics
- ETL storage
