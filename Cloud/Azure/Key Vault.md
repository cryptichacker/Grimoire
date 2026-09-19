---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Key Vault

## Up
- [[Azure]]

**Azure Key Vault** — Securely stores and controls access to secrets, keys and certificates, backed by HSMs.

**Category:** Identity & Security

## Key concepts
- Secrets, keys and certificates
- Access via RBAC or access policies
- Soft-delete and purge protection
- HSM-backed keys (Managed HSM)
- Key rotation
- Private endpoints

## Common CLI
| Command | Description |
|---|---|
| `az keyvault create` | Create a vault |
| `az keyvault secret set` | Store a secret |
| `az keyvault secret show` | Retrieve a secret |
| `az keyvault key create` | Create a key |

## Pricing model
Per operation for secrets/keys; per certificate; Managed HSM per hour.

## Works well with
- App Service, AKS, VMs
- Managed Identities
- Disk/Storage encryption

## Use cases
- Secret management
- Encryption keys
- TLS certificate storage
