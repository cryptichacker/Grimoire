---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT Core

## Up
- [[AWS]]

**AWS IoT Core** — Connects billions of devices to AWS and routes messages between them and cloud services securely at scale.

**Category:** IoT

## Key concepts
- MQTT/HTTP/WebSocket messaging
- Device registry (things) and Device Shadows
- X.509 certificate auth and policies
- Rules engine routing to AWS services
- Fleet indexing

## Common CLI
| Command | Description |
|---|---|
| `aws iot create-thing` | Register a device |
| `aws iot create-keys-and-certificate` | Issue device certificates |
| `aws iot-data publish` | Publish an MQTT message |

## Pricing model
Per million messages plus connectivity, rules and Device Shadow operations.

## Works well with
- Lambda, DynamoDB
- Kinesis
- IoT Greengrass

## Use cases
- Device connectivity
- Telemetry ingestion
- Command and control
