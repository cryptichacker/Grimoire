---
tags: [cloud, aws, game-tech]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Braket

## Up
- [[AWS]]

**Amazon Braket** — A managed quantum computing service to explore, build and run quantum algorithms on simulators and real QPUs.

**Category:** Game Tech

## Key concepts
- Quantum tasks
- Simulators (state vector/tensor)
- Access to third-party QPUs
- Hybrid jobs (classical + quantum)
- Braket SDK

## Common CLI
| Command | Description |
|---|---|
| `aws braket search-devices` | List quantum devices/simulators |
| `aws braket create-quantum-task` | Submit a quantum task |
| `aws braket get-device` | Get device details |

## Pricing model
Per task/shot on QPUs and per simulator-minute; hybrid jobs billed for compute.

## Works well with
- S3 (results)
- IAM
- SageMaker (notebooks)

## Use cases
- Quantum research
- Algorithm experimentation
- Hybrid quantum-classical apps
