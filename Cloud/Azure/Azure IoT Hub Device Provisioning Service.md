---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure IoT Hub Device Provisioning Service

## Up
- [[Azure]]

**Azure IoT Hub Device Provisioning Service (DPS)** — A helper service for IoT Hub that enables zero-touch, just-in-time provisioning of devices to the right hub at scale.

**Category:** IoT

## Key concepts
- Enrollment groups and individual enrollments
- Attestation (TPM, X.509, symmetric key)
- Allocation policies
- Multi-hub load balancing
- Reprovisioning

## Common CLI
| Command | Description |
|---|---|
| `az iot dps create` | Create a DPS instance |
| `az iot dps enrollment create` | Create an enrollment |
| `az iot dps linked-hub create` | Link an IoT hub |

## Pricing model
Per device provisioned (operations-based).

## Works well with
- IoT Hub
- IoT Edge
- Device Update

## Use cases
- Zero-touch provisioning
- Fleet onboarding at scale
- Multi-hub allocation
