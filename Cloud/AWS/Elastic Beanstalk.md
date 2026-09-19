---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Elastic Beanstalk

## Up
- [[AWS]]

**AWS Elastic Beanstalk** — Upload your code and Beanstalk provisions and manages the EC2, load balancer, scaling and health monitoring for you.

**Category:** Compute

## Key concepts
- Application vs Environment
- Platforms (Node, Python, Java, .NET, Docker...)
- Web vs Worker environment tiers
- Configuration via .ebextensions
- Rolling and blue/green deployments
- Managed platform updates

## Common CLI
| Command | Description |
|---|---|
| `eb init` | Set up the project |
| `eb create` | Create an environment |
| `eb deploy` | Deploy a new version |
| `eb status / eb logs` | Inspect status and logs |

## Pricing model
No charge for Beanstalk itself; pay for the underlying EC2, ELB and other resources.

## Works well with
- EC2, Elastic Load Balancing
- Auto Scaling
- RDS, CloudWatch

## Use cases
- Fast web-app deployment
- Teams wanting PaaS simplicity
- Standardized environments
