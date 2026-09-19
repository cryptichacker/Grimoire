---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Table Storage

## Up
- [[Azure]]

**Azure Table Storage** — A NoSQL key-value/attribute store for large amounts of semi-structured, schemaless data.

**Category:** Storage

## Key concepts
- Tables, entities and properties
- Partition and row keys
- Schemaless design
- OData queries
- Part of a storage account
- Cosmos DB Table API upgrade path

## Common CLI
| Command | Description |
|---|---|
| `az storage table create` | Create a table |
| `az storage entity insert` | Insert an entity |
| `az storage entity query` | Query entities |
| `az storage table list` | List tables |

## Pricing model
Per GB-month plus transactions.

## Works well with
- Functions, App Service
- Cosmos DB (Table API)

## Use cases
- Key-value data
- Device/metadata catalogs
- Cheap NoSQL storage
