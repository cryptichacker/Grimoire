---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CodeGuru

## Up
- [[AWS]]

**Amazon CodeGuru** — Uses ML to find code issues and costly lines: automated code reviews (Reviewer) and runtime performance profiling (Profiler).

**Category:** Developer Tools

## Key concepts
- CodeGuru Reviewer (PR/code analysis)
- CodeGuru Profiler (runtime profiling)
- Security detectors
- Recommendations
- Repository associations

## Common CLI
| Command | Description |
|---|---|
| `aws codeguru-reviewer associate-repository` | Connect a repo for reviews |
| `aws codeguru-reviewer list-recommendations` | List review findings |
| `aws codeguruprofiler create-profiling-group` | Create a profiling group |

## Pricing model
Per 100 lines analyzed (Reviewer) and per sampled hour (Profiler).

## Works well with
- CodeCommit / GitHub
- CodePipeline
- Lambda / EC2 (profiling)

## Use cases
- Automated code review
- Security detection in code
- Performance/cost profiling
