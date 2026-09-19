---
tags: [cloud, aws, end-user-computing]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# WorkSpaces Secure Browser

## Up
- [[AWS]]

**Amazon WorkSpaces Secure Browser** — A managed, fully isolated browser that streams web content so corporate data never lives on the user's device.

**Category:** End User Computing

## Key concepts
- Web portals
- Fully isolated browsing sessions
- No data on the endpoint
- Policy controls (copy/paste, printing)
- Identity federation

## Common CLI
| Command | Description |
|---|---|
| `aws workspaces-web create-portal` | Create a web portal |
| `aws workspaces-web create-browser-settings` | Configure browser policy |
| `aws workspaces-web list-portals` | List portals |

## Pricing model
Per active user per month.

## Works well with
- IAM Identity Center / IdPs
- VPC
- CloudWatch

## Use cases
- Secure access to internal web apps
- BYOD/contractor browsing
- Data-loss prevention
