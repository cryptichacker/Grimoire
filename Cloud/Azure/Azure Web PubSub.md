---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Web PubSub

## Up
- [[Azure]]

**Azure Web PubSub** — A managed service for building real-time messaging web apps using WebSockets and the publish/subscribe pattern.

**Category:** Integration

## Key concepts
- Hubs, groups and users
- Native WebSocket clients
- Pub/sub messaging
- Event handlers (serverless)
- Client protocols (JSON/protobuf)

## Common CLI
| Command | Description |
|---|---|
| `az webpubsub create` | Create a Web PubSub resource |
| `az webpubsub list` | List resources |
| `az webpubsub key show` | Get access keys |

## Pricing model
Per unit per day (free tier available).

## Works well with
- Functions (event handlers)
- App Service
- API Management

## Use cases
- Real-time web apps
- Live feeds/streaming
- IoT dashboards
