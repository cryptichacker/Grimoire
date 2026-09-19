---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Managed HSM

## Up
- [[Azure]]

**Azure Key Vault Managed HSM** — A fully managed, single-tenant, FIPS 140-3 Level 3 hardware security module for high-assurance key management.

**Category:** Identity & Security

## Key concepts
- Single-tenant HSM pools
- Customer-controlled keys
- Security domain and roles (local RBAC)
- BYOK import
- FIPS 140-3 Level 3
- Confidential-computing use cases

## Common CLI
| Command | Description |
|---|---|
| `az keyvault create --hsm-name` | Create a Managed HSM |
| `az keyvault key create --hsm-name` | Create an HSM-backed key |
| `az keyvault role assignment create` | Assign HSM roles |

## Pricing model
Per HSM instance-hour.

## Works well with
- Key Vault
- Disk/Storage encryption (CMK)
- Confidential computing

## Use cases
- High-assurance key custody
- Regulatory/FIPS requirements
- Sovereign key control
