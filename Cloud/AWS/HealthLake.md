---
tags: [cloud, aws, industry]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# HealthLake

## Up
- [[AWS]]

**AWS HealthLake** — A HIPAA-eligible service to store, transform, query and analyze health data at scale in the FHIR standard.

**Category:** Industry

## Key concepts
- FHIR data stores
- Import/export from S3
- Integrated NLP on medical text
- Standardized querying
- Analytics-ready output

## Common CLI
| Command | Description |
|---|---|
| `aws healthlake create-fhir-datastore` | Create a FHIR data store |
| `aws healthlake start-fhir-import-job` | Import health data |
| `aws healthlake describe-fhir-datastore` | Data store details |

## Pricing model
Per GB stored, data processed and NLP usage.

## Works well with
- S3
- Comprehend Medical
- QuickSight, Athena

## Use cases
- FHIR data lakes
- Population-health analytics
- Clinical data interoperability
