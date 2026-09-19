---
tags: [cloud, azure, hybrid]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Stack Edge

## Up
- [[Azure]]

**Azure Stack Edge** — A cloud-managed edge appliance that brings compute, storage and hardware-accelerated ML inference to on-prem/edge sites.

**Category:** Hybrid

## Key concepts
- Managed edge appliance (ordered from Azure)
- Local compute (VMs/containers) and storage gateway
- GPU/FPGA ML acceleration
- Data transfer to Azure
- Arc-manageable

## Common CLI
| Command | Description |
|---|---|
| `az databoxedge device list` | List Stack Edge devices |
| `az databoxedge device show` | Show device details |
| `Portal: order a device` | Order and configure the appliance |

## Pricing model
Monthly device subscription plus Azure services used.

## Works well with
- Blob Storage
- IoT Edge / AKS
- Machine Learning (edge inference)

## Use cases
- Edge ML inference
- Local data processing
- Cloud-to-edge data transfer
