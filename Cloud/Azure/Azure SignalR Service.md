---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure SignalR Service

## Up
- [[Azure]]

**Azure SignalR Service** — A managed service to add real-time web functionality (server-to-client push) to applications over WebSockets.

**Category:** Integration

## Key concepts
- Hubs and connections
- Default vs serverless mode
- Scale-out without managing WebSockets
- ASP.NET Core SignalR compatible
- Auto-scaling units

## Common CLI
| Command | Description |
|---|---|
| `az signalr create` | Create a SignalR resource |
| `az signalr list` | List resources |
| `az signalr key list` | Get access keys |

## Pricing model
Per unit per day (free tier available).

## Works well with
- Functions (serverless)
- App Service
- Web PubSub (alternative)

## Use cases
- Real-time dashboards
- Chat and notifications
- Live collaboration
