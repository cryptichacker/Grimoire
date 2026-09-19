---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Signer

## Up
- [[AWS]]

**AWS Signer** — Fully managed code-signing service to ensure the trust and integrity of your code and software artifacts.

**Category:** Security

## Key concepts
- Signing profiles
- Signing jobs
- Supported targets (Lambda, container images, IoT, code)
- Signature validation
- Revocation

## Common CLI
| Command | Description |
|---|---|
| `aws signer put-signing-profile` | Create a signing profile |
| `aws signer start-signing-job` | Sign an artifact |
| `aws signer list-signing-jobs` | List signing jobs |

## Pricing model
Per signing operation (some targets free).

## Works well with
- Lambda
- ECR (container images)
- IoT

## Use cases
- Code signing
- Software supply-chain integrity
- Trusted deployments
