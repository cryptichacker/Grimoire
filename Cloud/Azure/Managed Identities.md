---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Managed Identities

## Up
- [[Azure]]

**Azure Managed Identities** — Gives Azure resources an automatically managed identity in Entra ID to access other services without storing credentials.

**Category:** Identity & Security

## Key concepts
- System-assigned vs user-assigned
- No credentials in code
- Token acquisition via IMDS
- RBAC role assignments
- Federated identity credentials (workload identity)

## Common CLI
| Command | Description |
|---|---|
| `az identity create` | Create a user-assigned identity |
| `az identity list` | List identities |
| `az vm identity assign` | Assign identity to a VM |
| `az role assignment create` | Grant it access |

## Pricing model
Free.

## Works well with
- Key Vault
- Storage, SQL
- App Service, AKS, Functions

## Use cases
- Credential-free access
- Secure service-to-service auth
- Least-privilege automation
