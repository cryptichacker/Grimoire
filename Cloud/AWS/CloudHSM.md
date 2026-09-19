---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudHSM

## Up
- [[AWS]]

**AWS CloudHSM** — Managed, dedicated, single-tenant hardware security modules (FIPS 140-3) for full control of your encryption keys.

**Category:** Security

## Key concepts
- HSM clusters
- Single-tenant, customer-owned keys
- PKCS#11, JCE, CNG/KSP
- FIPS 140-3 validated
- No AWS access to key material

## Common CLI
| Command | Description |
|---|---|
| `aws cloudhsmv2 create-cluster` | Create an HSM cluster |
| `aws cloudhsmv2 create-hsm` | Add an HSM |
| `aws cloudhsmv2 describe-clusters` | List clusters |

## Pricing model
Per HSM instance-hour.

## Works well with
- KMS (custom key store)
- EC2 (client)
- ACM Private CA

## Use cases
- Dedicated key custody
- Regulatory/FIPS requirements
- Custom PKI and signing
