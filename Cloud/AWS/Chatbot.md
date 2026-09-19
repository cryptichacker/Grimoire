---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Chatbot

## Up
- [[AWS]]

**AWS Chatbot** — Lets you monitor and operate AWS resources from Slack and Microsoft Teams, receiving alerts and running commands.

**Category:** Management

## Key concepts
- Slack/Teams channel configurations
- Notifications from SNS
- Run CLI commands (with guardrails)
- IAM permissions per channel
- Custom actions

## Common CLI
| Command | Description |
|---|---|
| `aws chatbot create-slack-channel-configuration` | Connect a Slack channel |
| `aws chatbot describe-slack-channel-configurations` | List configs |
| `aws chatbot create-microsoft-teams-channel-configuration` | Connect Teams |

## Pricing model
Free (pay for underlying SNS/CloudWatch).

## Works well with
- SNS, CloudWatch
- Systems Manager
- Lambda

## Use cases
- ChatOps
- Alert routing to Slack/Teams
- Operating AWS from chat
