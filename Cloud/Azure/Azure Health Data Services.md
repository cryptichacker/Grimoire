---
tags: [cloud, azure, industry]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Health Data Services

## Up
- [[Azure]]

**Azure Health Data Services** — A managed set of health-data services (FHIR, DICOM, MedTech) to unify protected health information (PHI) in the cloud.

**Category:** Industry

## Key concepts
- Workspaces
- FHIR service (health records)
- DICOM service (medical imaging)
- MedTech service (device data)
- HIPAA/HITRUST eligible
- Interoperability standards

## Common CLI
| Command | Description |
|---|---|
| `az healthcareapis workspace create` | Create a workspace |
| `az healthcareapis workspace fhir-service create` | Create a FHIR service |
| `az healthcareapis workspace dicom-service create` | Create a DICOM service |

## Pricing model
Per storage, throughput and requests by service.

## Works well with
- Data Lake Storage
- AI Health Insights
- Synapse / Fabric

## Use cases
- FHIR data platform
- Medical imaging (DICOM)
- Device (IoT) health data
