---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Notification Hubs

## Up
- [[Azure]]

**Azure Notification Hubs** — A massively scalable mobile push-notification engine that pushes to any platform from any backend.

**Category:** Integration

## Key concepts
- Namespaces and hubs
- Platform Notification Systems (APNs, FCM, etc.)
- Tags and templates
- Broadcast to millions
- Registration management

## Common CLI
| Command | Description |
|---|---|
| `az notification-hub namespace create` | Create a namespace |
| `az notification-hub create` | Create a hub |
| `az notification-hub credential apns update` | Configure APNs |

## Pricing model
Per namespace tier plus pushes beyond the included quota.

## Works well with
- App Service / Functions
- Mobile apps
- Event Grid

## Use cases
- Mobile push notifications
- Cross-platform messaging
- Broadcast campaigns
