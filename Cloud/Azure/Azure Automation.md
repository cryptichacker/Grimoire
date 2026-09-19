---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Automation

## Up
- [[Azure]]

**Azure Automation** — Process automation, configuration management and update management for cloud and hybrid environments.

**Category:** Management

## Key concepts
- Runbooks (PowerShell/Python)
- Schedules and webhooks
- Hybrid Runbook Worker
- Shared assets (variables/credentials)
- State configuration (DSC)
- Update management (now via Azure Update Manager)

## Common CLI
| Command | Description |
|---|---|
| `az automation account create` | Create an automation account |
| `az automation runbook create` | Create a runbook |
| `az automation runbook start` | Run a runbook |

## Pricing model
Per job-minute of runbook execution and configuration nodes.

## Works well with
- Azure Monitor
- Managed Identities
- Logic Apps

## Use cases
- Ops automation
- Scheduled tasks
- Configuration management
