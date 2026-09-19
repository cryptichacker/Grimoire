---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EKS

## Up
- [[AWS]]

**Amazon Elastic Kubernetes Service** — Managed Kubernetes: AWS runs the highly available control plane while you run workloads on managed node groups or Fargate.

**Category:** Compute

## Key concepts
- Managed control plane
- Managed node groups and Fargate profiles
- IAM auth via aws-auth / access entries
- IRSA — IAM roles for service accounts
- Add-ons: VPC CNI, CoreDNS, kube-proxy
- Autoscaling with Cluster Autoscaler or Karpenter

## Common CLI
| Command | Description |
|---|---|
| `aws eks create-cluster` | Create a cluster |
| `aws eks update-kubeconfig` | Configure kubectl |
| `aws eks list-clusters` | List clusters |
| `aws eks describe-cluster` | Show cluster details |

## Pricing model
Hourly charge per cluster plus the cost of worker nodes (EC2 or Fargate).

## Works well with
- EC2, Fargate, ECR
- IAM, VPC, Elastic Load Balancing
- [[Karpenter]] for node autoscaling

## Use cases
- Portable Kubernetes workloads
- Hybrid and multi-cloud
- Platform engineering
