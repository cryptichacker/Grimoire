---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# GitHub Advanced Security for Azure DevOps

## Up
- [[Azure]]

**GitHub Advanced Security for Azure DevOps** — Brings GitHub Advanced Security code-scanning capabilities natively into Azure DevOps repositories and pipelines.

**Category:** Developer Tools

## Key concepts
- Secret scanning (push protection)
- Dependency scanning
- Code scanning (CodeQL)
- Alerts in the repo
- Pipeline integration

## Common CLI
| Command | Description |
|---|---|
| `Azure DevOps: enable Advanced Security` | Turn on per repo/project |
| `Pipeline task: AdvancedSecurity` | Run scans in CI |

## Pricing model
Per active committer per month.

## Works well with
- Azure Repos / Pipelines
- Microsoft Defender
- Azure Boards

## Use cases
- Secret detection
- Dependency/code scanning
- DevSecOps in Azure DevOps
