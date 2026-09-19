---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Sphere

## Up
- [[Azure]]

**Azure Sphere** — An end-to-end IoT security solution combining a secured microcontroller (MCU), OS and a cloud security service.

**Category:** IoT

## Key concepts
- Secured MCU chips
- Azure Sphere OS
- Security Service (updates/attestation)
- Certificate-based auth
- Over-the-air updates

## Common CLI
| Command | Description |
|---|---|
| `azsphere product create` | Create a product (Azure Sphere CLI) |
| `azsphere device show-attached` | Inspect a device |
| `azsphere image-package build` | Build an image |

## Pricing model
Per-device one-time licensing fee (includes ongoing security service).

## Works well with
- IoT Hub / IoT Central
- Device Provisioning Service
- OTA updates

## Use cases
- Secured connected devices
- Brownfield device security
- Long-lifecycle IoT
