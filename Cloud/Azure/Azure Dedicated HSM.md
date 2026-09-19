---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Dedicated HSM

## Up
- [[Azure]]

**Azure Dedicated HSM** — Provides physical, single-tenant HSM appliances in Azure for full administrative control of your keys.

**Category:** Identity & Security

## Key concepts
- Dedicated physical HSM (Thales Luna)
- Full customer administration
- FIPS 140-2 Level 3
- Deployed into your VNet
- Lift-and-shift PKCS#11/JCE apps

## Common CLI
| Command | Description |
|---|---|
| `az dedicated-hsm create` | Provision a dedicated HSM |
| `az dedicated-hsm list` | List HSMs |
| `az dedicated-hsm show` | Show HSM details |

## Pricing model
Per HSM appliance-hour.

## Works well with
- Virtual Network
- Key Vault (alternative)
- PKI / signing apps

## Use cases
- Full-control key custody
- Legacy HSM lift-and-shift
- Strict compliance
