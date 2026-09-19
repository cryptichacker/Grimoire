---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT Device Management

## Up
- [[AWS]]

**AWS IoT Device Management** — Onboard, organize, monitor and remotely manage large fleets of IoT devices throughout their lifecycle.

**Category:** IoT

## Key concepts
- Bulk registration and fleet provisioning
- Thing groups and dynamic groups
- Jobs for remote operations/OTA
- Secure tunneling
- Fleet indexing and search

## Common CLI
| Command | Description |
|---|---|
| `aws iot create-thing-group` | Create a thing group |
| `aws iot create-job` | Create a remote job |
| `aws iot create-provisioning-template` | Fleet provisioning template |

## Pricing model
Per bulk registration, remote action, indexing and tunneling usage.

## Works well with
- IoT Core
- S3 (OTA firmware)
- CloudWatch

## Use cases
- Fleet provisioning
- OTA firmware updates
- Remote device operations
