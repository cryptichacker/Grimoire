---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Lex

## Up
- [[AWS]]

**Amazon Lex** — Builds conversational interfaces (chatbots and voice bots) using the same technology as Alexa.

**Category:** Machine Learning

## Key concepts
- Bots, intents and slots
- Utterances and slot types
- Automatic speech recognition + NLU
- Fulfillment via Lambda
- Multi-language and channels

## Common CLI
| Command | Description |
|---|---|
| `aws lexv2-models create-bot` | Create a bot |
| `aws lexv2-models build-bot-locale` | Build a locale |
| `aws lexv2-runtime recognize-text` | Send text to the bot |

## Pricing model
Per text request or speech request processed.

## Works well with
- Lambda
- Amazon Connect
- Polly / Comprehend

## Use cases
- Chatbots
- IVR voice bots
- Self-service assistants
