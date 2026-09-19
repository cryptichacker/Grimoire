---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT Greengrass

## Up
- [[AWS]]

**AWS IoT Greengrass** — Edge runtime that brings local compute, messaging, ML inference and data sync to IoT devices, even offline.

**Category:** IoT

## Key concepts
- Greengrass core devices
- Components and deployments
- Local Lambda/containers
- Local pub/sub and shadows
- Offline operation and stream manager

## Common CLI
| Command | Description |
|---|---|
| `aws greengrassv2 create-component-version` | Publish a component |
| `aws greengrassv2 create-deployment` | Deploy to devices |
| `aws greengrassv2 list-core-devices` | List edge devices |

## Pricing model
Per active Greengrass core device per month.

## Works well with
- IoT Core
- Lambda
- SageMaker (ML at edge)

## Use cases
- Edge compute
- Local ML inference
- Intermittent-connectivity devices
