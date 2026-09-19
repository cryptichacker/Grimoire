---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EBS

## Up
- [[AWS]]

**Amazon Elastic Block Store** — Network-attached block volumes for EC2 that persist independently of the instance lifecycle.

**Category:** Storage

## Key concepts
- Volume types: gp3/gp2 (SSD), io1/io2 (provisioned IOPS), st1/sc1 (HDD)
- Snapshots stored in S3
- Encryption via KMS
- Multi-Attach (io2) for shared volumes
- Elastic Volumes — resize/change type live
- Volumes are Availability-Zone bound

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-volume` | Create a volume |
| `aws ec2 attach-volume` | Attach to an instance |
| `aws ec2 create-snapshot` | Snapshot a volume |
| `aws ec2 modify-volume` | Resize or change type |

## Pricing model
Per GB-month provisioned (plus IOPS/throughput for some types); snapshots billed per GB.

## Works well with
- EC2
- KMS
- Data Lifecycle Manager, AWS Backup

## Use cases
- Instance boot disks
- Databases on EC2
- Persistent application data
