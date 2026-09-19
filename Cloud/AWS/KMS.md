---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# KMS

## Up
- [[AWS]]

**AWS Key Management Service** — Managed service to create and control cryptographic keys used to encrypt data across AWS and your own applications.

**Category:** Security

## Key concepts
- Customer master keys (symmetric/asymmetric)
- AWS-managed vs customer-managed keys
- Key policies and grants
- Envelope encryption and data keys
- Automatic key rotation
- Multi-Region keys; CloudTrail auditing

## Common CLI
| Command | Description |
|---|---|
| `aws kms create-key` | Create a key |
| `aws kms encrypt / decrypt` | Encrypt or decrypt data |
| `aws kms generate-data-key` | Get a data key for envelope encryption |

## Pricing model
Per key per month plus per API request; many AWS-managed keys are free.

## Works well with
- S3, EBS, RDS
- Secrets Manager
- Virtually all services

## Use cases
- Encryption at rest
- Key governance
- Compliance
