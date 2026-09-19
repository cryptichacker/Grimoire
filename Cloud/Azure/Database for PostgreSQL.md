---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Database for PostgreSQL

## Up
- [[Azure]]

**Azure Database for PostgreSQL** — Fully managed PostgreSQL (Flexible Server) with high availability, scaling and automated maintenance.

**Category:** Database

## Key concepts
- Flexible Server deployment
- Zone-redundant high availability
- Read replicas
- Automated backups and PITR
- Burstable/General/Memory-optimized tiers
- Extensions

## Common CLI
| Command | Description |
|---|---|
| `az postgres flexible-server create` | Create a server |
| `az postgres flexible-server db create` | Create a database |
| `az postgres flexible-server list` | List servers |
| `az postgres flexible-server parameter set` | Tune parameters |

## Pricing model
Per vCore/hour and storage; stop/start to save on non-prod.

## Works well with
- Entra ID auth
- VNet integration
- App Service, AKS

## Use cases
- PostgreSQL workloads
- Web/app backends
- Open-source stacks
