---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Anomaly Detector

## Up
- [[Azure]]

**Azure AI Anomaly Detector** — An API that detects anomalies in time-series data automatically, without requiring machine-learning expertise.

**Category:** AI

## Key concepts
- Univariate and multivariate detection
- Batch and streaming detection
- Automatic model selection
- Sensitivity tuning
- Change-point detection

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind AnomalyDetector` | Create an Anomaly Detector resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per transaction/API call.

## Works well with
- Stream Analytics / Event Hubs
- IoT Hub
- Functions

## Use cases
- Anomaly detection
- IoT/telemetry monitoring
- Fraud/quality signals
