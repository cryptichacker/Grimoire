---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Database for MySQL

## Up
- [[Azure]]

**Azure Database for MySQL** — Fully managed MySQL (Flexible Server) with high availability, scaling and automated maintenance.

**Category:** Database

## Key concepts
- Flexible Server deployment
- Zone-redundant HA
- Read replicas
- Automated backups and PITR
- Burstable/General/Memory-optimized tiers
- Data-in replication

## Common CLI
| Command | Description |
|---|---|
| `az mysql flexible-server create` | Create a server |
| `az mysql flexible-server db create` | Create a database |
| `az mysql flexible-server list` | List servers |
| `az mysql flexible-server parameter set` | Tune parameters |

## Pricing model
Per vCore/hour and storage; stop/start to save on non-prod.

## Works well with
- Entra ID auth
- VNet integration
- App Service, AKS

## Use cases
- MySQL workloads
- LAMP/WordPress apps
- Open-source stacks
