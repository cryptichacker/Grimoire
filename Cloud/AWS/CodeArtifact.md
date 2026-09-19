---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CodeArtifact

## Up
- [[AWS]]

**AWS CodeArtifact** — Fully managed artifact repository for storing and sharing software packages (npm, PyPI, Maven, NuGet and more).

**Category:** Developer Tools

## Key concepts
- Domains and repositories
- Upstream repositories and external connections
- Package versions
- IAM-based access
- Retention

## Common CLI
| Command | Description |
|---|---|
| `aws codeartifact create-domain` | Create a domain |
| `aws codeartifact create-repository` | Create a repository |
| `aws codeartifact get-authorization-token` | Authenticate a package client |

## Pricing model
Per GB stored plus requests and data transfer.

## Works well with
- CodeBuild
- IAM
- npm/pip/maven clients

## Use cases
- Private package hosting
- Dependency proxying
- Build supply-chain control
