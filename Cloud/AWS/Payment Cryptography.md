---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Payment Cryptography

## Up
- [[AWS]]

**AWS Payment Cryptography** — Provides payment-specific cryptographic operations and key management for card processing without managing HSMs.

**Category:** Security

## Key concepts
- Payment HSM operations as a service
- PCI PIN/P2PE workflows
- Key import/export (TR-31/TR-34)
- Encrypt/decrypt/translate PINs
- Elastic and multi-AZ

## Common CLI
| Command | Description |
|---|---|
| `aws payment-cryptography create-key` | Create a payment key |
| `aws payment-cryptography-data encrypt-data` | Encrypt payment data |
| `aws payment-cryptography list-keys` | List keys |

## Pricing model
Per key-month plus per cryptographic API call.

## Works well with
- KMS
- Lambda
- Card processors

## Use cases
- Card payment processing
- PIN/P2PE operations
- PCI-compliant crypto
