---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Certificate Manager

## Up
- [[AWS]]

**AWS Certificate Manager** — Provision, manage and deploy free public TLS certificates (and private CA) for AWS services.

**Category:** Security

## Key concepts
- Public vs private certificates
- DNS or email validation
- Automatic renewal
- Managed private key with integration-only deployment
- ACM Private CA
- Regional service (CloudFront requires us-east-1)

## Common CLI
| Command | Description |
|---|---|
| `aws acm request-certificate` | Request a certificate |
| `aws acm list-certificates` | List certificates |
| `aws acm describe-certificate` | Show status/details |

## Pricing model
Public certificates are free; Private CA and issued private certs cost.

## Works well with
- CloudFront, Elastic Load Balancing
- API Gateway
- Route 53

## Use cases
- HTTPS for sites and APIs
- Auto-renewing TLS
- Internal PKI
