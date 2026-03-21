# Kanta Medication List — FHIR R4 Reference

Finnish Kanta medication list API returns medication data as FHIR R4 Bundles containing profiled resources with custom extensions.

## Profiles

| Profile | FHIR Resource | Description |
|---------|---------------|-------------|
| MedicationListMedicationRequest | MedicationRequest | Prescription data |
| MedicationListDiscontinuationMedicationRequest | MedicationRequest | Medication discontinuation record |
| MedicationListMedicationDispense | MedicationDispense | Dispensing data |
| MedicationListMedication | Medication | Medication details |
| MedicationListMedicationKnowledge | MedicationKnowledge | Extended medication info |
| MedicationListPatient | Patient | Patient data |
| MedicationListPractitioner | Practitioner | Prescriber data |
| MedicationListPractitionerRole | PractitionerRole | Prescriber role |
| MedicationListProvenance | Provenance | Audit/provenance of prescriptions |
| MedicationListOrganization | Organization | Organization data |
| MedicationListRenewalBasic | Basic | Renewal request and response |
| MedicationListStatusBasic | Basic | Lock and reservation status |
| MedicationListList | List | Groups MedicationRequest, MedicationDispense, and discontinuation records sharing the same active medication ID (KOL) and medication continuity sub-ID (JOT) |
| MedicationListDocumentReference | DocumentReference | Printout query response |

## Operations

All operations use HTTP POST with a `Parameters` resource as the request body.

### $get-medicationlist (Basic query)

Returns the current medication list for a patient. The response is a `Bundle` of type `searchset` containing profiled resources.

### $get-history-medications (Medication history)

Returns historical versions of a specific medication, identified by active medication ID.

### $get-discontinued-medications (Discontinued medications)

Returns medications that have been discontinued.

### $get-old-prescriptions (Historical prescriptions)

Returns historical prescriptions without active medication identifiers.

### $get-custom-data (Custom query)

Allows querying with custom filter parameters.

### $get-printout (Print query)

Returns a `DocumentReference` containing a printable representation.

## Extensions by Profile

### MedicationListMedicationRequest Extensions

| Extension | URL | Type | Description |
|-----------|-----|------|-------------|
| versionNumber | `http://resepti.kanta.fi/StructureDefinition/extension/versionNumber` | integer | Prescription version number |
| usage | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/usage` | CodeableConcept | Medication usage information |
| realDispenseStatus | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/realDispenseStatus` | CodeableConcept | Actual dispense status |
| dosageIfNeeded | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/dosageIfNeeded` | string | Conditional dosage instructions |
| medicinePauseInterval | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/medicinePauseInterval` | Period | Medicine pause interval |
| boundsDurationStartDate | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/boundsDurationStartDate` | dateTime | Start date for bounds duration |
| boundsRangeStartDate | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/boundsRangeStartDate` | dateTime | Start date for bounds range |

### MedicationListMedication Extensions

| Extension | URL | Type | Description |
|-----------|-----|------|-------------|
| pharmaceuticalProductStrength | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/pharmaceuticalProductStrength` | string | Product strength information |

### MedicationListMedicationKnowledge Extensions

| Extension | URL | Type | Description |
|-----------|-----|------|-------------|
| narcotic | `http://resepti.kanta.fi/fhir/StructureDefinition/extension/narcotic` | boolean | Whether the medication is a narcotic |

### Extension URL Patterns

Two naming conventions coexist:

- **Legacy**: `http://resepti.kanta.fi/StructureDefinition/extension/{name}` (e.g., `versionNumber`)
- **New**: `http://resepti.kanta.fi/fhir/StructureDefinition/extension/{name}` (e.g., `usage`, `realDispenseStatus`)

Both patterns are valid and used in production responses.

## Key Identifiers

- **KOL** (Käytössä olevan lääkkeen tunniste): Active medication identifier — groups all resources related to the same active medication
- **JOT** (Lääkejatkumon osatunniste): Medication continuity sub-identifier — tracks medication continuity

## Response Structure

A typical Kanta response Bundle contains:

```
Bundle (type: searchset)
├── MedicationRequest (prescription)
│   ├── contained: Medication (or referenced)
│   └── extensions: versionNumber, usage, realDispenseStatus, etc.
├── MedicationDispense (dispensing records)
│   └── contained: Medication (or referenced)
├── MedicationRequest (discontinuation, if applicable)
├── Patient
├── Practitioner / PractitionerRole
├── Organization
├── Provenance
├── List (grouping by KOL + JOT)
└── Basic (renewal requests, lock status)
```

Resources may use `contained` resources (referenced with `#` prefix) or be returned as separate Bundle entries with full references.

## Specification Links

- Implementation guide: https://simplifier.net/guide/finnish-kanta-medication-list-r4?version=current
- Extensions: https://simplifier.net/guide/Finnish-Kanta-Medication-list-R4/Home/Laajennokset?version=current
- Profiles: https://simplifier.net/guide/Finnish-Kanta-Medication-list-R4/Home/Profiilit?version=current
