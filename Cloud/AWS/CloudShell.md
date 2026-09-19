---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudShell

## Up
- [[AWS]]

**AWS CloudShell** — A browser-based shell, pre-authenticated with your console credentials, with AWS CLI and common tools preinstalled.

**Category:** Developer Tools

## Key concepts
- Launched from the AWS Console
- Pre-authenticated with your session
- 1 GB persistent home directory per Region
- AWS CLI v2, git, Python, Node preinstalled
- File upload/download

## Common CLI
| Command | Description |
|---|---|
| `aws --version` | AWS CLI v2 is preinstalled |
| `aws sts get-caller-identity` | Uses your console identity automatically |
| `Console -> CloudShell icon` | Launch the shell |

## Pricing model
Free (pay only for resources you create from it).

## Works well with
- IAM
- Every AWS service via CLI
- S3 (file transfer)

## Use cases
- Quick CLI tasks
- No-setup admin work
- Scripting from the browser
