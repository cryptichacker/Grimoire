---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Amazon MQ

## Up
- [[AWS]]

**Amazon MQ** — Managed message broker for Apache ActiveMQ and RabbitMQ, easing migration of apps that use standard messaging protocols.

**Category:** Application Integration

## Key concepts
- ActiveMQ and RabbitMQ engines
- Single-instance vs active/standby
- Standard protocols (JMS, AMQP, MQTT, STOMP)
- Broker configurations
- Encryption

## Common CLI
| Command | Description |
|---|---|
| `aws mq create-broker` | Create a broker |
| `aws mq list-brokers` | List brokers |
| `aws mq describe-broker` | Get broker details |

## Pricing model
Per broker-instance-hour plus storage.

## Works well with
- VPC
- CloudWatch
- Lambda

## Use cases
- Migrating existing messaging apps
- Standard-protocol messaging
- Hybrid integration
