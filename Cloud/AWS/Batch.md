---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Batch

## Up
- [[AWS]]

**AWS Batch** — Fully managed batch computing that dynamically provisions the optimal compute (EC2 or Fargate) to run large numbers of jobs.

**Category:** Compute

## Key concepts
- Compute environments (managed/unmanaged)
- Job queues and priorities
- Job definitions
- Array jobs and job dependencies
- Fair-share scheduling
- Spot integration

## Common CLI
| Command | Description |
|---|---|
| `aws batch create-compute-environment` | Create a compute environment |
| `aws batch register-job-definition` | Define a job |
| `aws batch submit-job` | Submit a job |
| `aws batch describe-jobs` | Check job status |

## Pricing model
No extra charge; pay for the EC2/Fargate resources the jobs consume.

## Works well with
- EC2, Fargate
- ECR
- CloudWatch

## Use cases
- HPC and scientific computing
- Rendering and transcoding
- Large data processing
