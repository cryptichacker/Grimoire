---
tags: [cloud, azure, integration]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Relay

## Up
- [[Azure]]

**Azure Relay** — Securely exposes services running on-premises to the cloud without opening firewall ports or changing the network.

**Category:** Integration

## Key concepts
- Hybrid Connections (WebSocket)
- WCF Relays
- Relay namespaces
- No inbound firewall ports
- Listener/sender model

## Common CLI
| Command | Description |
|---|---|
| `az relay namespace create` | Create a namespace |
| `az relay hyco create` | Create a Hybrid Connection |
| `az relay wcfrelay create` | Create a WCF relay |

## Pricing model
Per listener-hour plus messages/data.

## Works well with
- App Service (Hybrid Connections)
- On-prem services
- Service Bus

## Use cases
- Hybrid connectivity
- Exposing on-prem services
- Firewall-friendly integration
