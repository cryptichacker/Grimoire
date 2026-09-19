---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EFS

## Up
- [[AWS]]

**Amazon Elastic File System** — Elastic, shared NFS file system that grows and shrinks automatically and can be mounted by many instances at once.

**Category:** Storage

## Key concepts
- Mount targets per Availability Zone
- Performance modes (General Purpose / Max I/O)
- Throughput modes (Elastic / Provisioned / Bursting)
- Storage classes (Standard / IA) with lifecycle
- Access points and POSIX permissions
- Multi-AZ durability

## Common CLI
| Command | Description |
|---|---|
| `aws efs create-file-system` | Create a file system |
| `aws efs create-mount-target` | Create a mount target in a subnet |
| `aws efs create-access-point` | Create an access point |

## Pricing model
Per GB-month (Standard vs IA) plus throughput charges.

## Works well with
- EC2
- ECS/EKS
- Lambda, AWS Backup

## Use cases
- Shared content across servers
- Lift-and-shift of NFS workloads
- Container persistent storage
