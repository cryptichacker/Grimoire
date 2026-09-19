---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AWS Private CA

## Up
- [[AWS]]

**AWS Private Certificate Authority** — A managed private certificate authority to issue and manage private certificates for your internal PKI.

**Category:** Security

## Key concepts
- Root and subordinate CAs
- Certificate templates
- Certificate revocation (CRL/OCSP)
- Short-lived certificate mode
- Cross-account sharing

## Common CLI
| Command | Description |
|---|---|
| `aws acm-pca create-certificate-authority` | Create a private CA |
| `aws acm-pca issue-certificate` | Issue a certificate |
| `aws acm-pca get-certificate` | Retrieve a certificate |

## Pricing model
Per CA per month plus per certificate issued.

## Works well with
- Certificate Manager
- IAM Roles Anywhere
- EKS, IoT

## Use cases
- Internal TLS/PKI
- mTLS between services
- Device certificates
