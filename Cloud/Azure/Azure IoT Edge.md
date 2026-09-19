---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure IoT Edge

## Up
- [[Azure]]

**Azure IoT Edge** — Runs cloud workloads — containers, analytics and AI — directly on IoT edge devices, managed from IoT Hub.

**Category:** IoT

## Key concepts
- Edge runtime and modules
- Module deployment from the cloud
- Offline and store-and-forward
- Edge gateways
- Local ML inference
- Device twins

## Common CLI
| Command | Description |
|---|---|
| `az iot edge set-modules` | Deploy modules to a device |
| `az iot hub device-identity create --edge-enabled` | Register an edge device |
| `az iot edge deployment create` | Create an at-scale deployment |

## Pricing model
Free runtime; pay for IoT Hub and edge device hardware.

## Works well with
- IoT Hub
- Container Registry
- Machine Learning (edge models)

## Use cases
- Edge compute
- Local ML/AI inference
- Intermittent-connectivity devices
