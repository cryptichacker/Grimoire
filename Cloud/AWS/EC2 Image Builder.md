---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EC2 Image Builder

## Up
- [[AWS]]

**EC2 Image Builder** — Automates building, testing and distributing secure, up-to-date machine images (AMIs and container images).

**Category:** Compute

## Key concepts
- Image pipelines
- Image recipes and components
- Build and test stages
- Distribution across Regions/accounts
- Scheduled/triggered builds

## Common CLI
| Command | Description |
|---|---|
| `aws imagebuilder create-image-recipe` | Define an image recipe |
| `aws imagebuilder create-image-pipeline` | Create a pipeline |
| `aws imagebuilder create-image` | Build an image now |

## Pricing model
Free service; pay for the EC2/EBS used during builds and image storage.

## Works well with
- EC2
- ECR
- Systems Manager

## Use cases
- Golden AMI pipelines
- Hardened base images
- Automated image patching
