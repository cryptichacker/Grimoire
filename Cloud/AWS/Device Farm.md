---
tags: [cloud, aws, front-end-web-mobile]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Device Farm

## Up
- [[AWS]]

**AWS Device Farm** — Tests mobile and web apps on real devices in the AWS cloud, and enables remote device access.

**Category:** Front-End Web & Mobile

## Key concepts
- Real device fleet
- Automated test runs
- Supported frameworks (Appium, XCUITest, etc.)
- Remote access sessions
- Videos, logs and performance data

## Common CLI
| Command | Description |
|---|---|
| `aws devicefarm create-project` | Create a project |
| `aws devicefarm schedule-run` | Schedule a test run |
| `aws devicefarm list-runs` | List runs |

## Pricing model
Per device-minute, or a flat monthly unlimited-testing plan.

## Works well with
- CodePipeline
- S3
- CloudWatch

## Use cases
- Mobile app testing
- Cross-device QA
- Remote device debugging
