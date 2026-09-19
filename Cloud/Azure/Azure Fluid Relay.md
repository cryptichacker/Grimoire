---
tags: [cloud, azure, web]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Fluid Relay

## Up
- [[Azure]]

**Azure Fluid Relay** — A managed service powering the Fluid Framework for building real-time, multi-user collaborative applications.

**Category:** Web

## Key concepts
- Fluid Framework backend
- Real-time shared data structures
- Low-latency sync
- Session/relay management
- Client SDKs

## Common CLI
| Command | Description |
|---|---|
| `az fluid-relay server create` | Create a Fluid Relay server |
| `az fluid-relay server list` | List servers |
| `az fluid-relay server key list` | Get keys |

## Pricing model
By operations/messages and storage.

## Works well with
- Web / App Service
- SignalR / Web PubSub
- Entra ID

## Use cases
- Real-time collaboration
- Co-authoring apps
- Shared whiteboards/editors
