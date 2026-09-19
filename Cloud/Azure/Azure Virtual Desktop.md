---
tags: [cloud, azure, virtual-desktop]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Virtual Desktop

## Up
- [[Azure]]

**Azure Virtual Desktop (AVD)** — A desktop and app virtualization service (DaaS) delivering Windows desktops and apps from Azure.

**Category:** Virtual Desktop

## Key concepts
- Host pools (pooled/personal)
- Session hosts
- Application groups (desktop/RemoteApp)
- Workspaces
- FSLogix profile containers
- Multi-session Windows

## Common CLI
| Command | Description |
|---|---|
| `az desktopvirtualization hostpool create` | Create a host pool |
| `az desktopvirtualization workspace create` | Create a workspace |
| `az desktopvirtualization applicationgroup create` | Create an app group |

## Pricing model
Free control plane; pay for session-host VMs, storage and Windows licensing.

## Works well with
- Virtual Machines
- Entra ID / AD
- Azure Files (FSLogix)

## Use cases
- Virtual desktops
- RemoteApp delivery
- Secure remote work
