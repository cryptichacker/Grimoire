---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# App Runner

## Up
- [[AWS]]

**AWS App Runner** — Fully managed service to build, deploy and run containerized web apps and APIs directly from source or an image.

**Category:** Compute

## Key concepts
- Services from source (auto-build) or container image
- Automatic scaling and load balancing
- Managed TLS and custom domains
- VPC connector for private resources
- Auto-deploy on push

## Common CLI
| Command | Description |
|---|---|
| `aws apprunner create-service` | Create a service |
| `aws apprunner list-services` | List services |
| `aws apprunner start-deployment` | Trigger a deployment |

## Pricing model
Per provisioned and active compute/memory plus requests.

## Works well with
- ECR
- GitHub / source repos
- VPC, IAM

## Use cases
- Web apps and APIs
- Quick container deploys
- Teams avoiding orchestration
