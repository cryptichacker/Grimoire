---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Data Share

## Up
- [[Azure]]

**Azure Data Share** — A service to securely share data with external organizations, with control over terms and updates.

**Category:** Integration

## Key concepts
- Data shares (snapshot and in-place)
- Invitations and subscriptions
- Snapshot schedules
- Supported stores (Blob, ADLS, SQL, Synapse)
- Governed sharing terms

## Common CLI
| Command | Description |
|---|---|
| `az datashare account create` | Create a Data Share account |
| `az datashare create` | Create a share |
| `az datashare dataset create` | Add a dataset |

## Pricing model
Per snapshot execution (vCore-hours) plus storage/transfer.

## Works well with
- Data Lake / Blob Storage
- Azure SQL / Synapse
- Purview

## Use cases
- Cross-org data sharing
- Data monetization
- Partner data exchange
