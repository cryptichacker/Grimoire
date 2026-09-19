---
tags: [cloud, aws, end-user-computing]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AppStream 2.0

## Up
- [[AWS]]

**Amazon AppStream 2.0** — Streams individual desktop applications from the cloud to any device through a web browser.

**Category:** End User Computing

## Key concepts
- Fleets and stacks
- Application streaming (not full desktop)
- Image builder
- Elastic vs Always-On/On-Demand fleets
- User sessions and storage

## Common CLI
| Command | Description |
|---|---|
| `aws appstream create-fleet` | Create a fleet |
| `aws appstream create-stack` | Create a stack |
| `aws appstream create-streaming-url` | Generate a session URL |

## Pricing model
Per streaming instance-hour plus a per-user fee.

## Works well with
- S3 (home folders)
- Directory Service
- VPC

## Use cases
- App streaming to browsers
- SaaS-ifying desktop apps
- Training and demos
