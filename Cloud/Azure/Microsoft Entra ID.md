---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Entra ID

## Up
- [[Azure]]

**Microsoft Entra ID** — Cloud identity and access management (formerly Azure Active Directory) for authentication, SSO and authorization.

**Category:** Identity & Security

## Key concepts
- Users, groups and app registrations
- Single sign-on (SSO) and federation
- Conditional Access
- Multi-factor authentication
- Service principals and managed identities
- B2B and B2C

## Common CLI
| Command | Description |
|---|---|
| `az ad user create` | Create a user |
| `az ad group create` | Create a group |
| `az ad sp create-for-rbac` | Create a service principal |
| `az ad app create` | Register an application |

## Pricing model
Free tier plus Premium P1/P2 per user per month for advanced features.

## Works well with
- All Azure RBAC
- Microsoft 365
- App Service / AKS (workload identity)

## Use cases
- Identity and SSO
- Workforce/customer identity
- Conditional Access policies
