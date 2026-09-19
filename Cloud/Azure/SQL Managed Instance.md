---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# SQL Managed Instance

## Up
- [[Azure]]

**Azure SQL Managed Instance** — Managed SQL Server instance with near-100% engine compatibility for easy lift-and-shift.

**Category:** Database

## Key concepts
- Instance-level features (SQL Agent, cross-DB queries)
- vCore purchasing model
- VNet-native deployment
- Automated backups and PITR
- Failover groups
- Near-full SQL Server compatibility

## Common CLI
| Command | Description |
|---|---|
| `az sql mi create` | Create a managed instance |
| `az sql midb create` | Create a database |
| `az sql mi list` | List instances |
| `az sql mi update` | Update configuration |

## Pricing model
Per vCore/hour plus storage and backups; Azure Hybrid Benefit available.

## Works well with
- Virtual Network
- Entra ID
- Data Migration Service

## Use cases
- SQL Server migrations
- Instance-scoped features
- Enterprise databases
