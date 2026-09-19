---
tags: [cloud, aws, industry]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# HealthImaging

## Up
- [[AWS]]

**AWS HealthImaging** — A HIPAA-eligible service to store, transform and analyze medical imaging data (DICOM) at petabyte scale.

**Category:** Industry

## Key concepts
- Data stores for DICOM
- DICOM import jobs
- Image sets and frames
- Fast, cloud-native pixel access
- Metadata search

## Common CLI
| Command | Description |
|---|---|
| `aws medical-imaging create-datastore` | Create a data store |
| `aws medical-imaging start-dicom-import-job` | Import DICOM data |
| `aws medical-imaging get-image-set` | Retrieve an image set |

## Pricing model
Per GB stored plus data processed and retrieval.

## Works well with
- S3
- SageMaker
- HealthLake

## Use cases
- Medical image archives
- Imaging AI/ML pipelines
- PACS modernization
