---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudFront

## Up
- [[AWS]]

**Amazon CloudFront** — Global content delivery network that caches content at edge locations to reduce latency, with edge compute and security.

**Category:** Networking

## Key concepts
- Distributions and origins (S3/ALB/custom)
- Cache behaviors and TTLs
- Invalidations
- Origin Access Control for private S3
- CloudFront Functions and Lambda@Edge
- Signed URLs/cookies; WAF and Shield integration

## Common CLI
| Command | Description |
|---|---|
| `aws cloudfront create-distribution` | Create a distribution |
| `aws cloudfront create-invalidation` | Purge cached objects |
| `aws cloudfront list-distributions` | List distributions |

## Pricing model
Per GB of data transfer out and per request, varying by edge region tier.

## Works well with
- S3, Elastic Load Balancing
- ACM, WAF
- Route 53

## Use cases
- Website and asset delivery
- API acceleration
- Video streaming
