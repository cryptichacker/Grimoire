---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Secrets Manager

## Up
- [[AWS]]

**AWS Secrets Manager** — Securely stores, retrieves and automatically rotates secrets such as database credentials and API keys.

**Category:** Security

## Key concepts
- Secrets and versions
- Automatic rotation via Lambda
- Resource policies
- Cross-account and cross-region replication
- KMS encryption
- Retrieval through SDK/CLI

## Common CLI
| Command | Description |
|---|---|
| `aws secretsmanager create-secret` | Store a secret |
| `aws secretsmanager get-secret-value` | Retrieve a secret |
| `aws secretsmanager rotate-secret` | Rotate a secret |

## Pricing model
Per secret per month plus per 10,000 API calls.

## Works well with
- RDS / Aurora
- Lambda
- KMS, ECS

## Use cases
- Database credential rotation
- API keys
- Application secrets
