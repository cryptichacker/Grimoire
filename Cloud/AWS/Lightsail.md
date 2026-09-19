---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Lightsail

## Up
- [[AWS]]

**Amazon Lightsail** — Bundled, predictably priced virtual private servers (plus databases, load balancers and containers) aimed at simple projects.

**Category:** Compute

## Key concepts
- Fixed-price bundles
- Blueprints (WordPress, LAMP, Node...)
- Managed databases and load balancers
- Static IPs and snapshots
- Container service
- Clear upgrade path to EC2

## Common CLI
| Command | Description |
|---|---|
| `aws lightsail create-instances` | Create an instance |
| `aws lightsail get-instances` | List instances |
| `aws lightsail create-disk` | Add block storage |

## Pricing model
Flat monthly price per bundle.

## Works well with
- EC2 (for upgrades)
- Route 53
- S3

## Use cases
- Blogs and small websites
- Dev/test boxes
- Simple applications
