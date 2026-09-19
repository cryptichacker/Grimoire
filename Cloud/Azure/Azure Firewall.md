---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Firewall

## Up
- [[Azure]]

**Azure Firewall** — Cloud-native, managed, stateful network firewall with built-in high availability and scaling.

**Category:** Networking

## Key concepts
- Firewall policies
- Network and application rules
- Threat intelligence filtering
- DNAT for inbound
- IDPS (Premium)
- Forced tunneling

## Common CLI
| Command | Description |
|---|---|
| `az network firewall create` | Create a firewall |
| `az network firewall policy create` | Create a policy |
| `az network firewall network-rule create` | Add a network rule |

## Pricing model
Per deployment-hour plus per GB processed (Standard/Premium).

## Works well with
- Virtual Network / Hub
- Firewall Manager
- Route tables

## Use cases
- Central egress filtering
- Network segmentation
- Threat protection
