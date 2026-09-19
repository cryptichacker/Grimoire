---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Network Watcher

## Up
- [[Azure]]

**Azure Network Watcher** — A suite of tools to monitor, diagnose and gain insight into your Azure network.

**Category:** Networking

## Key concepts
- Connection Monitor
- IP flow verify and next hop
- NSG flow logs
- Packet capture
- Topology and diagnostics

## Common CLI
| Command | Description |
|---|---|
| `az network watcher configure` | Enable in a region |
| `az network watcher test-connectivity` | Test connectivity |
| `az network watcher flow-log create` | Enable NSG flow logs |

## Pricing model
Per check/monitor and per GB of logs/captures.

## Works well with
- Virtual Network, NSGs
- Log Analytics
- Load Balancer

## Use cases
- Network diagnostics
- Connectivity monitoring
- Traffic/flow analysis
