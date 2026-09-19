---
tags: [cloud, azure, web]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Static Web Apps

## Up
- [[Azure]]

**Azure Static Web Apps** — A service to build and deploy full-stack static web apps with globally distributed hosting and serverless APIs.

**Category:** Web

## Key concepts
- Global static hosting
- Built-in serverless APIs (Functions)
- GitHub/DevOps CI/CD
- Staging environments (per PR)
- Custom domains and free TLS
- Built-in auth

## Common CLI
| Command | Description |
|---|---|
| `az staticwebapp create` | Create a static web app |
| `az staticwebapp list` | List apps |
| `az staticwebapp environment list` | List environments |

## Pricing model
Free tier plus Standard per-app monthly with included bandwidth.

## Works well with
- Azure Functions (API)
- GitHub Actions / DevOps
- Front Door / DNS

## Use cases
- JAMstack/static sites
- SPA + serverless API
- Preview environments
