---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Confidential VMs

## Up
- [[Azure]]

**Azure Confidential VMs** — VMs that encrypt data in use with hardware-based trusted execution environments to protect data and code while processing.

**Category:** Compute

## Key concepts
- Hardware TEEs (AMD SEV-SNP / Intel TDX)
- Memory encryption in use
- VM guest attestation
- Confidential OS disk encryption
- Secure boot and vTPM

## Common CLI
| Command | Description |
|---|---|
| `az vm create --security-type ConfidentialVM` | Create a confidential VM |
| `az vm list` | List VMs |
| `az attestation ...` | Verify VM integrity |

## Pricing model
Standard VM pricing for confidential-capable sizes.

## Works well with
- Azure Attestation
- Key Vault / Managed HSM
- Confidential Ledger

## Use cases
- Protecting data in use
- Regulated/sensitive workloads
- Confidential computing
