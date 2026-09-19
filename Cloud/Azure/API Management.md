---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# API Management

## Up
- [[Azure]]

**Azure API Management** — Publishes, secures, transforms and monitors APIs through a managed gateway with a developer portal.

**Category:** Integration

## Key concepts
- Gateway, management and developer portal
- Products, APIs and operations
- Policies (rate limit, transform, auth)
- Subscriptions and API keys
- Tiers (Consumption to Premium)
- Self-hosted gateway

## Common CLI
| Command | Description |
|---|---|
| `az apim create` | Create an API Management service |
| `az apim api import` | Import an API (OpenAPI) |
| `az apim product create` | Create a product |
| `az apim api list` | List APIs |

## Pricing model
Per gateway-hour by tier (Consumption is per-call).

## Works well with
- Functions, App Service, AKS
- Entra ID
- Application Insights

## Use cases
- API gateways
- Securing/monetizing APIs
- Legacy API modernization
