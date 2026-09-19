---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Wavelength

## Up
- [[AWS]]

**AWS Wavelength** — Deploys compute and storage inside telecom 5G networks so applications reach mobile users with ultra-low latency.

**Category:** Compute

## Key concepts
- Wavelength Zones inside carrier networks
- Carrier gateway for mobile traffic
- Runs EC2/EBS/EKS at the edge
- Anchored to a parent Region

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 describe-availability-zones --filters Name=zone-type,Values=wavelength-zone` | List Wavelength Zones |
| `aws ec2 create-carrier-gateway` | Create a carrier gateway |

## Pricing model
Standard EC2/EBS pricing for resources in Wavelength Zones plus carrier data.

## Works well with
- EC2, EBS
- VPC
- EKS

## Use cases
- 5G/mobile edge apps
- AR/VR and gaming
- Real-time media processing
