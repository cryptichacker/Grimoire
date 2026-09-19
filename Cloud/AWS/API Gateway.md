---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# API Gateway

## Up
- [[AWS]]

**Amazon API Gateway** — Fully managed service to build, secure and operate REST, HTTP and WebSocket APIs at scale.

**Category:** Networking

## Key concepts
- REST vs HTTP vs WebSocket APIs
- Resources, methods and integrations
- Stages and deployments
- Authorizers (IAM/Cognito/Lambda)
- Usage plans, API keys and throttling
- Mapping templates and custom domains

## Common CLI
| Command | Description |
|---|---|
| `aws apigateway create-rest-api` | Create a REST API |
| `aws apigatewayv2 create-api` | Create an HTTP/WebSocket API |
| `aws apigateway create-deployment` | Deploy to a stage |

## Pricing model
Per million API calls plus data transfer/caching; HTTP APIs are cheaper than REST.

## Works well with
- Lambda
- Cognito, WAF
- CloudWatch, Step Functions

## Use cases
- Serverless APIs
- Backend front doors
- Webhooks
