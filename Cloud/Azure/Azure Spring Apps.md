---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Spring Apps

## Up
- [[Azure]]

**Azure Spring Apps** — A fully managed service for running Spring Boot and Spring Cloud microservices without managing infrastructure.

**Category:** Compute

## Key concepts
- Managed Spring runtime
- Service discovery and config server
- Blue-green deployments
- Autoscaling
- Tiers (Standard/Enterprise)
- Managed components (Tanzu)

## Common CLI
| Command | Description |
|---|---|
| `az spring create` | Create a Spring Apps instance |
| `az spring app create` | Create an app |
| `az spring app deploy` | Deploy code/jar |

## Pricing model
Per vCPU/memory-hour by tier.

## Works well with
- Application Insights
- Key Vault
- Azure SQL / Cosmos DB

## Use cases
- Spring microservices
- Java modernization
- Cloud-native Java apps
