---
tags: [cloud, aws, game-tech]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Ground Station

## Up
- [[AWS]]

**AWS Ground Station** — Fully managed ground-station-as-a-service to control satellites and ingest their data without owning antennas.

**Category:** Game Tech

## Key concepts
- Satellite onboarding
- Contacts (reserved antenna time)
- Mission profiles and dataflow endpoints
- Global antenna network
- Data delivery to VPC/S3

## Common CLI
| Command | Description |
|---|---|
| `aws groundstation list-satellites` | List onboarded satellites |
| `aws groundstation reserve-contact` | Reserve antenna time |
| `aws groundstation list-contacts` | List scheduled contacts |

## Pricing model
Per minute of antenna contact time.

## Works well with
- EC2 (processing)
- S3
- VPC

## Use cases
- Satellite downlink/command
- Earth-observation data
- Space data processing
