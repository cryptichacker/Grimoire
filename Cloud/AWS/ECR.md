---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# ECR

## Up
- [[AWS]]

**Amazon Elastic Container Registry** — Private and public container image registry integrated with IAM, ECS/EKS and vulnerability scanning.

**Category:** Compute

## Key concepts
- Repositories with tags and digests
- Lifecycle policies to expire old images
- Image scanning (basic and enhanced via Inspector)
- Cross-region and cross-account replication
- Stores OCI artifacts too

## Common CLI
| Command | Description |
|---|---|
| `aws ecr create-repository` | Create a repository |
| `aws ecr get-login-password` | Authenticate Docker to ECR |
| `aws ecr describe-images` | List images |
| `aws ecr put-lifecycle-policy` | Set cleanup rules |

## Pricing model
Storage per GB-month plus data transfer; enhanced scanning adds cost.

## Works well with
- ECS, EKS, Docker
- IAM
- Inspector

## Use cases
- Store application images
- CI/CD image pushes
- Image vulnerability scanning
