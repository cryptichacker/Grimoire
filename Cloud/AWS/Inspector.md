---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Inspector

## Up
- [[AWS]]

**Amazon Inspector** — Automated, continuous vulnerability management that scans EC2, ECR container images and Lambda for CVEs and network exposure.

**Category:** Security

## Key concepts
- Continuous automated scanning
- Software vulnerabilities (CVE) and network reachability
- Risk-scored findings
- Multi-account coverage
- SBOM export

## Common CLI
| Command | Description |
|---|---|
| `aws inspector2 enable` | Enable scanning |
| `aws inspector2 list-findings` | List findings |
| `aws inspector2 list-coverage` | See what is covered |

## Pricing model
Per scanned resource (instance, image, function) per month.

## Works well with
- EC2, ECR, Lambda
- Security Hub
- Organizations

## Use cases
- Vulnerability management
- Container image scanning
- Continuous compliance
