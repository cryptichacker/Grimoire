---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CodeBuild

## Up
- [[AWS]]

**AWS CodeBuild** — Fully managed continuous-integration service that compiles source, runs tests and produces deployable artifacts.

**Category:** Developer Tools

## Key concepts
- Build projects and buildspec.yml
- Managed or custom build images
- Compute types/environments
- Artifacts to S3
- Caching
- Batch builds

## Common CLI
| Command | Description |
|---|---|
| `aws codebuild create-project` | Create a build project |
| `aws codebuild start-build` | Run a build |
| `aws codebuild batch-get-builds` | Check build status |

## Pricing model
Per build minute by compute type.

## Works well with
- CodePipeline
- GitHub / source repos
- ECR, S3

## Use cases
- CI builds and tests
- Container image builds
- Artifact packaging
