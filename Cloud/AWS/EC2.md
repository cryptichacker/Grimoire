---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EC2

## Up
- [[AWS]]

**Amazon Elastic Compute Cloud** — Rent resizable virtual servers (instances) by the second, choosing an AMI, instance type, storage and network — the backbone of most AWS workloads.

**Category:** Compute

## Key concepts
- AMI (machine image) defines the OS/software baseline
- Instance families/types (t/m/c/r/g) trade off CPU, memory, GPU
- EBS-backed vs instance-store root volumes
- Security groups (stateful firewall) and key pairs (SSH)
- Elastic IPs and ENIs for networking
- Auto Scaling Groups + launch templates
- Purchase options: On-Demand, Spot, Reserved, Savings Plans

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 run-instances` | Launch one or more instances |
| `aws ec2 describe-instances` | List/inspect instances |
| `aws ec2 stop-instances / start-instances` | Stop or start |
| `aws ec2 terminate-instances` | Delete instances |
| `aws ec2 create-key-pair` | Create an SSH key pair |

## Pricing model
Per-second/hour by instance type and OS; large savings via Spot, Reserved Instances or Savings Plans.

## Works well with
- EBS, VPC, Elastic Load Balancing
- Auto Scaling, CloudWatch
- IAM instance roles

## Use cases
- Web and application servers
- Auto-scaling compute fleets
- Lift-and-shift of on-prem VMs
