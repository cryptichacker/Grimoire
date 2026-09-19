---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Microsoft Playwright Testing

## Up
- [[Azure]]

**Microsoft Playwright Testing** — A managed service that runs Playwright end-to-end browser tests at scale across parallel cloud browsers.

**Category:** Developer Tools

## Key concepts
- Cloud-hosted browsers
- Massively parallel test runs
- Cross-browser/OS coverage
- Uses your existing Playwright tests
- CI/CD integration

## Common CLI
| Command | Description |
|---|---|
| `npx playwright test --config=playwright.service.config.ts` | Run tests on the service |
| `Portal: create a workspace` | Provision the testing workspace |

## Pricing model
Per test-minute of parallel browser execution.

## Works well with
- Azure DevOps / GitHub Actions
- Static Web Apps / App Service
- Load Testing

## Use cases
- End-to-end testing
- Parallel browser tests
- CI/CD test acceleration
