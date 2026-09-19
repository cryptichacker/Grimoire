---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cloud Map

## Up
- [[AWS]]

**AWS Cloud Map** — Service discovery for cloud resources: register application components and discover them by name with health awareness.

**Category:** Networking

## Key concepts
- Namespaces (DNS and API)
- Services and service instances
- Custom attributes
- Health checking
- Used by ECS service discovery

## Common CLI
| Command | Description |
|---|---|
| `aws servicediscovery create-private-dns-namespace` | Create a namespace |
| `aws servicediscovery create-service` | Register a service |
| `aws servicediscovery register-instance` | Register an instance |

## Pricing model
Per registered resource plus discovery API calls (and Route 53 for DNS namespaces).

## Works well with
- ECS
- Route 53
- VPC

## Use cases
- Microservice discovery
- Dynamic endpoints
- Service registries
