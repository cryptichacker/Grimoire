---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Bastion

## Up
- [[Azure]]

**Azure Bastion** — Provides secure RDP and SSH access to VMs directly in the portal without exposing public IPs.

**Category:** Identity & Security

## Key concepts
- Browser-based RDP/SSH
- No public IP on VMs
- Deployed per VNet
- SKUs (Basic/Standard/Premium)
- Native client support
- Session recording (Premium)

## Common CLI
| Command | Description |
|---|---|
| `az network bastion create` | Create a Bastion host |
| `az network bastion rdp` | RDP to a VM |
| `az network bastion ssh` | SSH to a VM |

## Pricing model
Per Bastion-hour by SKU plus outbound data.

## Works well with
- Virtual Machines
- Virtual Network
- Entra ID

## Use cases
- Secure VM access
- Eliminating public IPs
- Jump-box replacement
