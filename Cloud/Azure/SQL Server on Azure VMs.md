---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# SQL Server on Azure VMs

## Up
- [[Azure]]

**SQL Server on Azure Virtual Machines** — Runs full SQL Server on Azure VMs (IaaS) with the SQL IaaS Agent extension for automated management.

**Category:** Database

## Key concepts
- SQL IaaS Agent extension
- Automated backups and patching
- Azure Hybrid Benefit (BYOL)
- Availability groups
- Storage configuration best practices

## Common CLI
| Command | Description |
|---|---|
| `az sql vm create` | Register/create a SQL Server VM |
| `az sql vm list` | List SQL VMs |
| `az sql vm update` | Update SQL VM configuration |

## Pricing model
Per VM compute plus SQL Server licensing (or Hybrid Benefit).

## Works well with
- Virtual Machines
- Managed Disks
- Azure Backup

## Use cases
- Full SQL Server control
- OS-level access
- Lift-and-shift SQL
