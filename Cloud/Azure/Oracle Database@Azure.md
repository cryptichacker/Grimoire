---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Oracle Database@Azure

## Up
- [[Azure]]

**Oracle Database@Azure** — Runs Oracle Database services (including Exadata and Autonomous Database) on OCI hardware co-located in Azure datacenters.

**Category:** Database

## Key concepts
- Oracle Exadata Database Service
- Autonomous Database
- Deployed in Azure regions
- Low-latency to Azure services
- Unified billing and support

## Common CLI
| Command | Description |
|---|---|
| `az oracle-database autonomous-database create` | Create an Autonomous Database |
| `az oracle-database cloud-vm-cluster create` | Create an Exadata VM cluster |
| `az oracle-database list` | List resources |

## Pricing model
Per Oracle service (OCPU/storage) via Azure Marketplace billing.

## Works well with
- Virtual Network
- Entra ID
- Azure apps/analytics

## Use cases
- Oracle workloads on Azure
- Exadata/Autonomous DB
- Oracle migrations
