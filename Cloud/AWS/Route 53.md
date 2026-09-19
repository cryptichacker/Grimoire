---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Route 53

## Up
- [[AWS]]

**Amazon Route 53** — Highly available DNS, domain registration and traffic routing with health checks and failover.

**Category:** Networking

## Key concepts
- Public and private hosted zones
- Record types (A/AAAA/CNAME/Alias)
- Routing policies: simple, weighted, latency, geolocation, failover, multivalue
- Health checks
- Alias records that point to AWS resources
- Domain registration

## Common CLI
| Command | Description |
|---|---|
| `aws route53 create-hosted-zone` | Create a hosted zone |
| `aws route53 change-resource-record-sets` | Add or update records |
| `aws route53 list-hosted-zones` | List zones |

## Pricing model
Per hosted zone per month, per million queries, and per health check.

## Works well with
- CloudFront, Elastic Load Balancing
- S3, ACM
- API Gateway

## Use cases
- DNS management
- Global traffic routing
- DR failover
