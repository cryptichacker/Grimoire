---
tags: [cloud, azure, storage]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Storage Actions

## Up
- [[Azure]]

**Azure Storage Actions** — A serverless service to automate data-management operations on Blob Storage objects at scale without code.

**Category:** Storage

## Key concepts
- Storage tasks
- Conditions and operations
- Assignments to accounts
- Scheduled or on-demand runs
- Reporting

## Common CLI
| Command | Description |
|---|---|
| `az storage-actions task create` | Create a storage task |
| `az storage-actions task list` | List tasks |
| `az storage-actions task assignment create` | Assign to an account |

## Pricing model
Per objects targeted/operated on.

## Works well with
- Blob Storage
- Lifecycle management
- Event Grid

## Use cases
- Bulk blob operations
- Tiering/tag automation
- Data-management at scale
