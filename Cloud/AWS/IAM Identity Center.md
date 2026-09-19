---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IAM Identity Center

## Up
- [[AWS]]

**AWS IAM Identity Center** — Centrally manages workforce access and single sign-on to multiple AWS accounts and business applications.

**Category:** Security

## Key concepts
- Identity source (built-in, AD, or external IdP)
- Permission sets
- Account assignments
- SSO access portal
- SCIM provisioning
- Integrated with Organizations

## Common CLI
| Command | Description |
|---|---|
| `aws sso-admin list-instances` | Find the Identity Center instance |
| `aws sso-admin create-permission-set` | Define an access level |
| `aws sso-admin create-account-assignment` | Grant access to an account |

## Pricing model
Free.

## Works well with
- Organizations
- IAM
- External IdPs and business apps

## Use cases
- Multi-account SSO
- Workforce access management
- Centralized permissions
