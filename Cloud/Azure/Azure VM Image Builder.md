---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure VM Image Builder

## Up
- [[Azure]]

**Azure VM Image Builder** — Automates building customized, hardened VM images from a template (built on HashiCorp Packer).

**Category:** Compute

## Key concepts
- Image templates
- Customizers (shell/PowerShell/Windows Update)
- Distribution (Managed Image / Shared Image Gallery)
- Source images
- Build automation

## Common CLI
| Command | Description |
|---|---|
| `az image builder create` | Create an image template |
| `az image builder run` | Build the image |
| `az image builder list` | List templates |

## Pricing model
Free service; pay for the compute/storage used during the build.

## Works well with
- Compute Gallery
- Virtual Machine Scale Sets
- Azure DevOps / GitHub

## Use cases
- Golden images
- Hardened base images
- Automated image pipelines
