---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Directory Service

## Up
- [[AWS]]

**AWS Directory Service** — Managed Microsoft Active Directory (and alternatives) to use with AWS resources and directory-aware workloads.

**Category:** Security

## Key concepts
- AWS Managed Microsoft AD
- AD Connector (proxy to on-prem)
- Simple AD
- Trust relationships
- Seamless domain join

## Common CLI
| Command | Description |
|---|---|
| `aws ds create-microsoft-ad` | Create a managed AD |
| `aws ds create-directory` | Create Simple AD |
| `aws ds describe-directories` | List directories |

## Pricing model
Per directory-hour by edition/size.

## Works well with
- WorkSpaces, RDS SQL Server
- IAM Identity Center
- EC2 (domain join)

## Use cases
- Active Directory in AWS
- Windows workload auth
- Hybrid identity
