---
tags: [cloud, aws, end-user-computing]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# WorkSpaces

## Up
- [[AWS]]

**Amazon WorkSpaces** — Fully managed, persistent virtual desktops (DaaS) for Windows and Linux, accessible from anywhere.

**Category:** End User Computing

## Key concepts
- Bundles (compute + storage)
- Persistent user desktops
- AlwaysOn vs AutoStop billing
- Directory integration
- Pools (non-persistent) option

## Common CLI
| Command | Description |
|---|---|
| `aws workspaces create-workspaces` | Provision desktops |
| `aws workspaces describe-workspaces` | List desktops |
| `aws workspaces terminate-workspaces` | Remove desktops |

## Pricing model
Monthly or hourly per WorkSpace by bundle.

## Works well with
- Directory Service
- IAM Identity Center
- VPC

## Use cases
- Remote/virtual desktops
- Contractor and BYOD access
- Secure workstations
