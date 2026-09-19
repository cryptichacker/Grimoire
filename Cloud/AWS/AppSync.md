---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AppSync

## Up
- [[AWS]]

**AWS AppSync** — Managed GraphQL (and Pub/Sub) service that connects apps to data sources with real-time and offline capabilities.

**Category:** Application Integration

## Key concepts
- GraphQL schema and resolvers
- Data sources (DynamoDB/Lambda/RDS/HTTP)
- Real-time subscriptions
- Caching
- Authorization modes (Cognito/IAM/API key/OIDC)
- Merged APIs

## Common CLI
| Command | Description |
|---|---|
| `aws appsync create-graphql-api` | Create an API |
| `aws appsync start-schema-creation` | Upload a schema |
| `aws appsync create-resolver` | Add a resolver |

## Pricing model
Per query/mutation and real-time update, plus caching.

## Works well with
- DynamoDB, Lambda
- Cognito
- RDS

## Use cases
- Mobile/web GraphQL backends
- Real-time apps
- Offline-capable apps
