---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# X-Ray

## Up
- [[AWS]]

**AWS X-Ray** — Distributed tracing service that helps analyze and debug the behavior and performance of applications and microservices.

**Category:** Developer Tools

## Key concepts
- Traces, segments and subsegments
- Service map
- Sampling rules
- Annotations and metadata
- Insights
- X-Ray SDK and OpenTelemetry

## Common CLI
| Command | Description |
|---|---|
| `aws xray get-trace-summaries` | List recent traces |
| `aws xray batch-get-traces` | Fetch full traces |
| `aws xray get-service-graph` | Get the service map |

## Pricing model
Per trace recorded, retrieved and scanned.

## Works well with
- Lambda, API Gateway
- ECS/EKS
- CloudWatch

## Use cases
- Latency debugging
- Distributed tracing
- Bottleneck analysis
