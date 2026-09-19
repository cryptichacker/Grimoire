---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Verified Permissions

## Up
- [[AWS]]

**Amazon Verified Permissions** — A scalable, fine-grained authorization service for your own applications using the Cedar policy language.

**Category:** Security

## Key concepts
- Policy stores
- Cedar policies and schema
- Is-authorized API
- Policy templates
- Identity source integration

## Common CLI
| Command | Description |
|---|---|
| `aws verifiedpermissions create-policy-store` | Create a policy store |
| `aws verifiedpermissions create-policy` | Add a policy |
| `aws verifiedpermissions is-authorized` | Make an authz decision |

## Pricing model
Per authorization request.

## Works well with
- Cognito
- API Gateway / Lambda
- Application backends

## Use cases
- App authorization
- Fine-grained access control
- Externalized permissions
