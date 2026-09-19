---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Systems Manager

## Up
- [[AWS]]

**AWS Systems Manager** — A suite for operating AWS and on-prem fleets: run commands, patch, manage parameters and open shells.

**Category:** Management

## Key concepts
- Managed instances via the SSM agent
- Run Command
- Session Manager (shell without SSH)
- Patch Manager
- Parameter Store
- State Manager, Automation runbooks, Inventory

## Common CLI
| Command | Description |
|---|---|
| `aws ssm send-command` | Run a command across instances |
| `aws ssm start-session` | Open a shell to an instance |
| `aws ssm put-parameter / get-parameter` | Store or read config/secrets |

## Pricing model
Many features are free; advanced parameters and automation can incur charges.

## Works well with
- EC2
- IAM, CloudWatch
- Secrets Manager, S3

## Use cases
- Fleet management
- Patch automation
- Config and secret storage
