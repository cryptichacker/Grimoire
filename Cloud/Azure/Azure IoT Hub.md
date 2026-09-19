---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure IoT Hub

## Up
- [[Azure]]

**Azure IoT Hub** — A managed service acting as a central message hub for bi-directional communication between IoT devices and the cloud.

**Category:** IoT

## Key concepts
- Device identity registry
- Device-to-cloud and cloud-to-device messaging
- Device twins
- Direct methods
- Message routing
- Device Provisioning Service (DPS)

## Common CLI
| Command | Description |
|---|---|
| `az iot hub create` | Create an IoT hub |
| `az iot hub device-identity create` | Register a device |
| `az iot device send-d2c-message` | Send a device message |

## Pricing model
Per hub unit (message tier) per day.

## Works well with
- IoT Central / Edge
- Event Hubs, Functions
- Stream Analytics, Digital Twins

## Use cases
- Device connectivity
- Telemetry ingestion
- Command and control
