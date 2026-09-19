---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Outposts

## Up
- [[AWS]]

**AWS Outposts** — Racks and servers that bring native AWS infrastructure and services on-premises for low-latency and data-residency needs.

**Category:** Compute

## Key concepts
- Outposts racks vs servers
- Local AWS services (EC2/EBS/S3 on Outposts)
- Anchored to a home Region
- Local gateway networking
- Capacity management

## Common CLI
| Command | Description |
|---|---|
| `aws outposts list-outposts` | List Outposts |
| `aws outposts get-outpost` | Get Outpost details |
| `aws outposts list-catalog-items` | Browse the hardware catalog |

## Pricing model
Purchase or subscription of hardware capacity (1 or 3 year terms).

## Works well with
- EC2, EBS, S3
- VPC
- ECS/EKS

## Use cases
- Low-latency on-prem workloads
- Data-residency requirements
- Hybrid cloud
