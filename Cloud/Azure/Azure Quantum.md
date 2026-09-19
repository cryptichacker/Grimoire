---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Quantum

## Up
- [[Azure]]

**Azure Quantum** — A cloud service providing access to quantum hardware, simulators and optimization solvers from multiple providers.

**Category:** Compute

## Key concepts
- Workspaces
- Quantum jobs and targets
- Q# and Qiskit/Cirq support
- Third-party QPUs (IonQ, Quantinuum, etc.)
- Resource estimation
- Hybrid quantum-classical

## Common CLI
| Command | Description |
|---|---|
| `az quantum workspace create` | Create a workspace |
| `az quantum target list` | List available targets |
| `az quantum job submit` | Submit a quantum job |

## Pricing model
Per QPU shot/usage and simulator time by provider.

## Works well with
- Storage (results)
- Azure Machine Learning
- VS Code / Q# SDK

## Use cases
- Quantum research
- Algorithm experimentation
- Resource estimation
