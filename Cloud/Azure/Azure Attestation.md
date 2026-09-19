---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Attestation

## Up
- [[Azure]]

**Azure Attestation** — Remotely verifies the trustworthiness of a platform and the integrity of binaries running inside trusted execution environments.

**Category:** Identity & Security

## Key concepts
- Attestation providers and policies
- TPM and SGX/enclave attestation
- Confidential VM attestation
- Signed attestation tokens
- Policy management

## Common CLI
| Command | Description |
|---|---|
| `az attestation create` | Create an attestation provider |
| `az attestation list` | List providers |
| `az attestation show` | Show a provider |

## Pricing model
Per attestation operation.

## Works well with
- Confidential Computing / Confidential VMs
- Key Vault / Managed HSM
- Confidential Ledger

## Use cases
- TEE attestation
- Confidential-computing trust
- Platform integrity verification
