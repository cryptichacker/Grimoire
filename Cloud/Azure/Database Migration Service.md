---
tags: [cloud, azure, migration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Database Migration Service

## Up
- [[Azure]]

**Azure Database Migration Service** — A managed service to migrate databases to Azure data platforms with minimal downtime, online or offline.

**Category:** Migration

## Key concepts
- Migration projects and tasks
- Online (continuous) vs offline
- Homogeneous and heterogeneous migrations
- Assessment via Data Migration Assistant
- Schema and data migration

## Common CLI
| Command | Description |
|---|---|
| `az dms create` | Create a DMS instance |
| `az dms project create` | Create a migration project |
| `az dms project task create` | Create a migration task |

## Pricing model
Standard tier free; Premium per compute-hour.

## Works well with
- Azure SQL / PostgreSQL / MySQL
- Azure Migrate
- Virtual Network

## Use cases
- Database migrations
- Near-zero-downtime cutover
- Engine modernization
