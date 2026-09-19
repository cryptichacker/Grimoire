---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure SQL Database

## Up
- [[Azure]]

**Azure SQL Database** — Fully managed, intelligent relational database based on the latest SQL Server engine (PaaS).

**Category:** Database

## Key concepts
- Single database vs elastic pools
- Purchasing models (DTU vs vCore)
- Serverless and Hyperscale tiers
- Automatic backups and PITR
- Active geo-replication and failover groups
- Built-in intelligence/tuning

## Common CLI
| Command | Description |
|---|---|
| `az sql server create` | Create a logical server |
| `az sql db create` | Create a database |
| `az sql db list` | List databases |
| `az sql db copy` | Copy a database |

## Pricing model
Per DTU or vCore/hour by tier (serverless bills per second of compute).

## Works well with
- Entra ID auth
- Key Vault (TDE)
- App Service, Functions

## Use cases
- Transactional apps
- SaaS multi-tenant DBs
- Managed SQL without ops
