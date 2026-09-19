---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Defender for Cloud

## Up
- [[Azure]]

**Microsoft Defender for Cloud** — Cloud security posture management (CSPM) and workload protection (CWPP) across Azure, hybrid and multicloud.

**Category:** Identity & Security

## Key concepts
- Secure Score and recommendations
- Regulatory compliance dashboard
- Defender plans per workload
- Attack path analysis
- Just-in-time VM access
- Multicloud (AWS/GCP) connectors

## Common CLI
| Command | Description |
|---|---|
| `az security pricing list` | Show Defender plan status |
| `az security assessment list` | List posture findings |
| `az security alert list` | List security alerts |

## Pricing model
Free CSPM baseline; per-resource Defender plans for workload protection.

## Works well with
- Microsoft Sentinel
- Azure Policy
- Log Analytics

## Use cases
- Security posture management
- Workload threat protection
- Compliance reporting
