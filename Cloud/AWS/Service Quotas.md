---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Service Quotas

## Up
- [[AWS]]

**AWS Service Quotas** — A central place to view and manage your AWS service quotas (limits) and request increases.

**Category:** Management

## Key concepts
- Quotas per service/Region
- Increase requests
- Quota utilization via CloudWatch
- Organization templates
- Default vs applied quotas

## Common CLI
| Command | Description |
|---|---|
| `aws service-quotas list-service-quotas` | List quotas for a service |
| `aws service-quotas request-service-quota-increase` | Request an increase |
| `aws service-quotas get-service-quota` | Get a specific quota |

## Pricing model
Free.

## Works well with
- CloudWatch (alarms)
- Organizations
- Trusted Advisor

## Use cases
- Quota visibility
- Proactive limit increases
- Avoiding throttling
