---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Sentinel

## Up
- [[Azure]]

**Microsoft Sentinel** — Cloud-native SIEM and SOAR that collects, detects, investigates and responds to threats at scale.

**Category:** Identity & Security

## Key concepts
- Data connectors
- Analytics rules and detections
- Incidents and investigation
- Workbooks
- Automation (playbooks via Logic Apps)
- UEBA and threat hunting

## Common CLI
| Command | Description |
|---|---|
| `az sentinel data-connector list` | List data connectors |
| `az sentinel alert-rule list` | List analytics rules |
| `az sentinel incident list` | List incidents |

## Pricing model
Per GB of data ingested/analyzed (commitment tiers available).

## Works well with
- Log Analytics workspace
- Defender for Cloud
- Logic Apps (playbooks)

## Use cases
- SIEM/SOAR
- Threat detection
- Incident response
