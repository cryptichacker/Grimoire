---
tags: [cloud, aws, migration-transfer]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Transfer Family

## Up
- [[AWS]]

**AWS Transfer Family** — Fully managed SFTP, FTPS, FTP and AS2 file transfer directly into and out of S3 and EFS.

**Category:** Migration & Transfer

## Key concepts
- Protocols: SFTP, FTPS, FTP, AS2
- Managed or custom identity providers
- Endpoints (public/VPC)
- Workflows for post-upload processing
- Managed file transfers

## Common CLI
| Command | Description |
|---|---|
| `aws transfer create-server` | Create a transfer server |
| `aws transfer create-user` | Create a user |
| `aws transfer list-servers` | List servers |

## Pricing model
Per protocol-enabled endpoint-hour plus per-GB uploaded/downloaded.

## Works well with
- S3, EFS
- IAM / directory services
- Lambda (workflows)

## Use cases
- B2B file exchange
- SFTP into S3
- Partner data ingestion
