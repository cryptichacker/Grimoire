---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cognito

## Up
- [[AWS]]

**Amazon Cognito** — Adds sign-up, sign-in and access control to apps, plus identity federation and temporary AWS credentials.

**Category:** Security

## Key concepts
- User pools (authentication and directory)
- Identity pools (federated AWS credentials)
- Hosted UI
- OAuth2/OIDC/SAML federation
- MFA and Lambda triggers
- ID, access and refresh tokens

## Common CLI
| Command | Description |
|---|---|
| `aws cognito-idp create-user-pool` | Create a user pool |
| `aws cognito-idp create-user-pool-client` | Create an app client |
| `aws cognito-identity create-identity-pool` | Create an identity pool |

## Pricing model
Per monthly active user with a free tier; advanced security features cost extra.

## Works well with
- API Gateway, ALB
- AppSync, Lambda
- IAM

## Use cases
- App authentication
- Social and enterprise login
- Mobile identity
