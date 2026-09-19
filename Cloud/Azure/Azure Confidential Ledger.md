---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Confidential Ledger

## Up
- [[Azure]]

**Azure Confidential Ledger** — A tamper-proof, cryptographically verifiable ledger for sensitive data records, running in confidential enclaves.

**Category:** Database

## Key concepts
- Append-only ledger
- Cryptographic receipts and proof
- Runs in hardware-backed TEEs (SGX)
- Blockchain-based integrity
- Merkle-tree verification

## Common CLI
| Command | Description |
|---|---|
| `az confidentialledger create` | Create a ledger |
| `az confidentialledger list` | List ledgers |
| `az confidentialledger show` | Show a ledger |

## Pricing model
Per ledger instance plus transactions/storage.

## Works well with
- Confidential Computing
- Key Vault / Managed HSM
- Storage

## Use cases
- Tamper-evident records
- Audit and compliance logs
- High-integrity data
