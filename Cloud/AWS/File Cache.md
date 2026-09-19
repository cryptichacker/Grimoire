---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# File Cache

## Up
- [[AWS]]

**Amazon File Cache** — A fully managed, high-speed cache for datasets stored across on-prem NFS and S3, giving unified low-latency access.

**Category:** Storage

## Key concepts
- High-speed Lustre-based cache
- Links to S3 and NFS sources
- Lazy loading and write-back
- Temporary/burst workloads

## Common CLI
| Command | Description |
|---|---|
| `aws fsx create-file-cache` | Create a file cache |
| `aws fsx describe-file-caches` | List caches |
| `aws fsx delete-file-cache` | Delete a cache |

## Pricing model
Per GB-month of cache capacity plus throughput.

## Works well with
- S3
- FSx for Lustre
- EC2 / EKS

## Use cases
- Hybrid data processing
- Burst compute over dispersed data
- HPC data staging
