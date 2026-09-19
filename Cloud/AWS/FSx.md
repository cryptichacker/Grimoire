---
tags: [cloud, aws, storage]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# FSx

## Up
- [[AWS]]

**Amazon FSx** — Fully managed third-party file systems: Windows File Server, Lustre, NetApp ONTAP and OpenZFS.

**Category:** Storage

## Key concepts
- FSx for Windows (SMB, Active Directory)
- FSx for Lustre (HPC, S3-linked)
- FSx for NetApp ONTAP
- FSx for OpenZFS
- Backups and Multi-AZ options

## Common CLI
| Command | Description |
|---|---|
| `aws fsx create-file-system` | Create a file system |
| `aws fsx describe-file-systems` | List file systems |
| `aws fsx create-backup` | Create a backup |

## Pricing model
Per GB-month by file-system type plus throughput/IOPS and backups.

## Works well with
- EC2
- Directory Service
- S3 (Lustre), AWS Backup

## Use cases
- Windows shared storage
- HPC scratch storage
- Enterprise NAS
