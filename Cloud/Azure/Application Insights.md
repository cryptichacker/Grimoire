---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Application Insights

## Up
- [[Azure]]

**Azure Application Insights** — Application performance monitoring (APM) for live web apps — requests, dependencies, exceptions and traces.

**Category:** Management

## Key concepts
- Auto-instrumentation and SDK
- Application Map
- Live Metrics
- Distributed tracing
- Availability tests
- Usage analytics; workspace-based

## Common CLI
| Command | Description |
|---|---|
| `az monitor app-insights component create` | Create a resource |
| `az monitor app-insights query` | Query telemetry |
| `az monitor app-insights events show` | Inspect events |

## Pricing model
Per GB of telemetry ingested (billed via Log Analytics).

## Works well with
- Azure Monitor / Log Analytics
- App Service, Functions, AKS
- OpenTelemetry

## Use cases
- APM and tracing
- Performance debugging
- Availability monitoring
