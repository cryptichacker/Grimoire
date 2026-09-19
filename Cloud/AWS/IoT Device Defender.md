---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT Device Defender

## Up
- [[AWS]]

**AWS IoT Device Defender** — Audits IoT configurations and continuously monitors device fleets to detect anomalies and security issues.

**Category:** IoT

## Key concepts
- Audit checks against best practices
- Rules-based and ML detect (security profiles)
- Behavior anomaly detection
- Alarms and mitigation actions
- Fleet metrics

## Common CLI
| Command | Description |
|---|---|
| `aws iot create-security-profile` | Create a security profile |
| `aws iot create-scheduled-audit` | Schedule an audit |
| `aws iot list-active-violations` | List violations |

## Pricing model
Per device per month for monitoring plus per audit check.

## Works well with
- IoT Core
- IoT Device Management
- SNS, CloudWatch

## Use cases
- IoT security monitoring
- Fleet compliance audits
- Anomaly detection
