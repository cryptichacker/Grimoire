---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Device Update for IoT Hub

## Up
- [[Azure]]

**Azure Device Update for IoT Hub** — A service to publish, target and deploy over-the-air (OTA) updates to IoT and edge devices at scale.

**Category:** IoT

## Key concepts
- Update accounts and instances
- Update manifests and groups
- Phased/gradual rollouts
- Delta updates
- Compliance and status reporting

## Common CLI
| Command | Description |
|---|---|
| `az iot du account create` | Create a Device Update account |
| `az iot du instance create` | Create an instance |
| `az iot du update import` | Import an update |

## Pricing model
By device/update operations.

## Works well with
- IoT Hub
- IoT Edge
- Device Provisioning Service

## Use cases
- OTA firmware updates
- Fleet update management
- Secure device patching
