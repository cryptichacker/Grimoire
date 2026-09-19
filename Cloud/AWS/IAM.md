---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IAM

## Up
- [[AWS]]

**AWS Identity and Access Management** — Controls who is authenticated and authorized for AWS resources through users, groups, roles and policies.

**Category:** Security

## Key concepts
- Users, groups and roles
- Policy types: identity, resource, SCP
- Principle of least privilege
- Roles and AssumeRole via STS
- Instance profiles for EC2
- MFA, access keys, permission boundaries
- IAM Identity Center for workforce SSO

## Common CLI
| Command | Description |
|---|---|
| `aws iam create-user / create-role` | Create principals |
| `aws iam attach-role-policy` | Grant permissions |
| `aws iam create-policy` | Create a custom policy |
| `aws iam list-users` | List users |

## Pricing model
Free.

## Works well with
- Every AWS service
- STS
- Organizations, IAM Identity Center

## Use cases
- Access control
- Cross-account access
- Federated login
