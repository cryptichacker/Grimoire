---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Snow Family

## Up
- [[AWS]]

**AWS Snow Family** — Physical devices (Snowball Edge, Snowcone) for offline data transfer and edge compute where the network is limited.

**Category:** Storage

## Key concepts
- Snowball Edge (storage/compute optimized)
- Snowcone (small, rugged)
- NFS and S3-compatible interfaces
- Edge compute (EC2/Lambda)
- Encryption; ordered via console

## Common CLI
| Command | Description |
|---|---|
| `aws snowball create-job` | Create a device job |
| `aws snowball describe-job` | Check job status |
| `aws snowball list-jobs` | List jobs |

## Pricing model
Per-device job fee plus days of use and data transfer.

## Works well with
- S3
- EC2 (edge)
- IAM/KMS

## Use cases
- Large offline migrations
- Disconnected/edge locations
- DR data shipping
