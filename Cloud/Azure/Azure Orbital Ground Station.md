---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Orbital Ground Station

## Up
- [[Azure]]

**Azure Orbital Ground Station** — A ground-station-as-a-service to communicate with, downlink from and command satellites without owning antennas.

**Category:** Networking

## Key concepts
- Spacecraft registration
- Contact scheduling (antenna time)
- Contact profiles and endpoints
- Global antenna partner network
- Data delivery to Azure

## Common CLI
| Command | Description |
|---|---|
| `az orbital spacecraft create` | Register a spacecraft |
| `az orbital contact create` | Schedule a contact |
| `az orbital contact-profile create` | Create a contact profile |

## Pricing model
Per minute of antenna contact time.

## Works well with
- Virtual Network
- Storage / Event Hubs
- VMs (processing)

## Use cases
- Satellite downlink/command
- Earth-observation data
- Space data pipelines
