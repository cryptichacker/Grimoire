---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Load Testing

## Up
- [[Azure]]

**Azure Load Testing** — A fully managed load-testing service to generate high-scale load and find performance bottlenecks.

**Category:** Developer Tools

## Key concepts
- Test runs and test plans
- Apache JMeter and Locust support
- Client-side and server-side metrics
- CI/CD integration
- Thresholds and pass/fail criteria
- Multi-region load

## Common CLI
| Command | Description |
|---|---|
| `az load create` | Create a load-testing resource |
| `az load test create` | Create a test |
| `az load test-run create` | Run a load test |

## Pricing model
Per virtual-user-hour of generated load.

## Works well with
- Azure Monitor / App Insights
- Azure DevOps / GitHub Actions
- App Service, AKS

## Use cases
- Performance testing
- Scalability validation
- Regression in CI/CD
