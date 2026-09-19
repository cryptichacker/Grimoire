---
tags: [cloud, aws, industry]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# HealthOmics

## Up
- [[AWS]]

**AWS HealthOmics** — Helps healthcare and life-science organizations store, query and analyze genomic, transcriptomic and other omics data.

**Category:** Industry

## Key concepts
- Sequence and reference stores
- Variant and annotation stores
- Bioinformatics workflows (WDL/Nextflow/CWL)
- Run groups
- Analytics via Athena

## Common CLI
| Command | Description |
|---|---|
| `aws omics create-sequence-store` | Create a sequence store |
| `aws omics create-workflow` | Create a workflow |
| `aws omics start-run` | Run a bioinformatics workflow |

## Pricing model
Per GB stored, per compute-run and data processed.

## Works well with
- S3
- Athena
- SageMaker

## Use cases
- Genomics pipelines
- Omics data storage
- Bioinformatics analytics
