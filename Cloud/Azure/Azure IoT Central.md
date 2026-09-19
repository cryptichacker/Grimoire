---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure IoT Central

## Up
- [[Azure]]

**Azure IoT Central** — A managed app platform (aPaaS) to build and operate IoT solutions quickly with device templates and dashboards.

**Category:** IoT

## Key concepts
- Applications and device templates
- Rules and dashboards
- Data export
- Jobs for fleet operations
- Built on IoT Hub

## Common CLI
| Command | Description |
|---|---|
| `az iot central app create` | Create an IoT Central app |
| `az iot central app list` | List apps |
| `az iot central device create` | Create a device |

## Pricing model
Per device per month (with free messages allotment).

## Works well with
- IoT Hub (underlying)
- Event Hubs / Blob (export)
- Power BI

## Use cases
- Rapid IoT solutions
- Device management
- Operator dashboards
