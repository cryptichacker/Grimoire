---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Analysis Services

## Up
- [[Azure]]

**Azure Analysis Services** — An enterprise-grade analytical engine (tabular models) for building semantic models over data for BI.

**Category:** Analytics

## Key concepts
- Tabular semantic models
- DAX and MDX
- Partitions and processing
- Scale-out query replicas
- Row-level security

## Common CLI
| Command | Description |
|---|---|
| `az analysisservices server create` | Create a server |
| `az analysisservices server list` | List servers |
| `az analysisservices server suspend/resume` | Pause/resume |

## Pricing model
Per server-hour by tier (pause to save cost).

## Works well with
- Power BI
- Azure SQL / Synapse
- Data Lake Storage

## Use cases
- Semantic BI models
- Enterprise reporting
- Self-service analytics backends
