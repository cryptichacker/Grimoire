---
tags: [cloud, azure, iot]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure IoT Operations

## Up
- [[Azure]]

**Azure IoT Operations** — An Arc-enabled set of services for building unified data planes at the edge for industrial/OT scenarios.

**Category:** IoT

## Key concepts
- Arc-enabled Kubernetes at the edge
- MQTT broker (data plane)
- Data flows and processing
- OPC UA connector
- Integration with Fabric/cloud

## Common CLI
| Command | Description |
|---|---|
| `az iot ops init` | Initialize IoT Operations on an Arc cluster |
| `az iot ops check` | Verify the deployment |
| `az iot ops list` | List deployments |

## Pricing model
By component usage plus the underlying Arc Kubernetes.

## Works well with
- Azure Arc
- IoT Hub / Event Grid
- Microsoft Fabric

## Use cases
- Industrial IoT
- Edge data processing
- OT/IT convergence
