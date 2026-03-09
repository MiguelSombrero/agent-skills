---
name: fhir-specialist
description: Senior FHIR developer specializing in Spring Boot applications with HAPI FHIR R4 client integrations. Expertise in Kanta medication list API, FHIR resource extraction, and clean Java code with Stream API. Use when developing FHIR integrations, connecting to FHIR APIs, parsing FHIR Bundles, working with MedicationRequest/MedicationDispense resources, or integrating with Finnish Kanta services.
---

# FHIR Specialist

Senior developer with comprehensive knowledge of the HL7 FHIR R4 specification, specialized in building Spring Boot applications that integrate with FHIR APIs using the HAPI FHIR Java client.

## Core Principles

- Use HAPI FHIR Generic (Fluent) Client for all FHIR API interactions
- Extract data from FHIR resources using Java Stream API for clean, readable code
- Use `FhirContext.forR4()` — share a single `FhirContext` instance (expensive to create, thread-safe)
- Client instances are lightweight and can be created per-request
- Always handle FHIR extensions explicitly when working with profiled resources (especially Kanta)
- Prefer strongly-typed resource access over raw JSON parsing

## HAPI FHIR Client Setup

### Dependencies (Maven)

```xml
<dependency>
    <groupId>ca.uhn.hapi.fhir</groupId>
    <artifactId>hapi-fhir-client</artifactId>
</dependency>
<dependency>
    <groupId>ca.uhn.hapi.fhir</groupId>
    <artifactId>hapi-fhir-structures-r4</artifactId>
</dependency>
```

### Spring Configuration

```java
@Configuration
public class FhirConfig {

    @Bean
    public FhirContext fhirContext() {
        return FhirContext.forR4();
    }

    @Bean
    public IGenericClient fhirClient(FhirContext fhirContext,
                                      @Value("${fhir.server.base-url}") String baseUrl) {
        fhirContext.getRestfulClientFactory().setSocketTimeout(30_000);
        return fhirContext.newRestfulGenericClient(baseUrl);
    }
}
```

## Invoking FHIR Operations

### Extended Operations (e.g., Kanta $get-medicationlist)

```java
Parameters inParams = new Parameters();
inParams.addParameter().setName("paramName").setValue(new StringType("value"));

Parameters outParams = client.operation()
        .onType("MedicationRequest")
        .named("$get-medicationlist")
        .withParameters(inParams)
        .execute();
```

### Search Operations

```java
Bundle results = client.search()
        .forResource(MedicationRequest.class)
        .where(MedicationRequest.PATIENT.hasId("patient-id"))
        .returnBundle(Bundle.class)
        .execute();
```

## Extracting Resources from Bundles

Use Java Stream API to extract typed resources from FHIR Bundles:

```java
public <T extends IBaseResource> List<T> extractResources(Bundle bundle, Class<T> type) {
    return bundle.getEntry().stream()
            .map(Bundle.BundleEntryComponent::getResource)
            .filter(type::isInstance)
            .map(type::cast)
            .toList();
}
```

### Extracting specific data from resources

```java
List<String> medicationNames = extractResources(bundle, MedicationRequest.class).stream()
        .map(MedicationRequest::getMedicationReference)
        .map(Reference::getDisplay)
        .filter(Objects::nonNull)
        .toList();
```

## Working with FHIR Extensions

Extensions carry additional data beyond the base FHIR resource. Access them by URL:

```java
public Optional<String> getExtensionStringValue(DomainResource resource, String url) {
    return resource.getExtension().stream()
            .filter(ext -> url.equals(ext.getUrl()))
            .map(Extension::getValue)
            .filter(StringType.class::isInstance)
            .map(v -> ((StringType) v).getValue())
            .findFirst();
}

public Optional<Extension> getExtension(DomainResource resource, String url) {
    return resource.getExtension().stream()
            .filter(ext -> url.equals(ext.getUrl()))
            .findFirst();
}
```

## Resolving Contained and Referenced Resources

FHIR resources can reference other resources via `Reference` fields. In Kanta responses, resources may be contained within the parent resource or returned as separate Bundle entries.

```java
public Optional<Medication> resolveMedication(MedicationRequest request, Bundle bundle) {
    Reference medicationRef = request.getMedicationReference();
    if (medicationRef.getReference().startsWith("#")) {
        return request.getContained().stream()
                .filter(Medication.class::isInstance)
                .map(Medication.class::cast)
                .filter(m -> m.getId().equals(medicationRef.getReference()))
                .findFirst();
    }
    return extractResources(bundle, Medication.class).stream()
            .filter(m -> m.getIdElement().toUnqualifiedVersionless().getValue()
                    .equals(medicationRef.getReference()))
            .findFirst();
}
```

## Kanta Medication List API

Finnish Kanta medication list API uses FHIR R4 with custom profiles and extensions. For detailed profile/extension reference, see [kanta-reference.md](kanta-reference.md).

### Key Operations

| Operation                       | Purpose                                                |
| ------------------------------- | ------------------------------------------------------ |
| `$get-medicationlist`           | Basic medication list query                            |
| `$get-history-medications`      | Medication history query                               |
| `$get-discontinued-medications` | Discontinued medications query                         |
| `$get-old-prescriptions`        | Historical prescriptions without active medication IDs |
| `$get-custom-data`              | Custom data query                                      |
| `$get-printout`                 | Print output query                                     |

### Key Kanta Extension URLs

```java
public static final String EXT_VERSION_NUMBER =
    "http://resepti.kanta.fi/StructureDefinition/extension/versionNumber";
public static final String EXT_USAGE =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/usage";
public static final String EXT_REAL_DISPENSE_STATUS =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/realDispenseStatus";
public static final String EXT_DOSAGE_IF_NEEDED =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/dosageIfNeeded";
public static final String EXT_MEDICINE_PAUSE_INTERVAL =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/medicinePauseInterval";
public static final String EXT_BOUNDS_DURATION_START_DATE =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/boundsDurationStartDate";
public static final String EXT_BOUNDS_RANGE_START_DATE =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/boundsRangeStartDate";
public static final String EXT_PHARMACEUTICAL_PRODUCT_STRENGTH =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/pharmaceuticalProductStrength";
public static final String EXT_NARCOTIC =
    "http://resepti.kanta.fi/fhir/StructureDefinition/extension/narcotic";
```

Note: Older extensions use the pattern `http://resepti.kanta.fi/StructureDefinition/extension/...` while newer ones use `http://resepti.kanta.fi/fhir/StructureDefinition/extension/...`.

## Error Handling

Always handle FHIR-specific exceptions:

```java
try {
    Bundle result = client.search()
            .forResource(MedicationRequest.class)
            .returnBundle(Bundle.class)
            .execute();
} catch (ResourceNotFoundException e) {
    // 404 - resource not found
} catch (FhirClientConnectionException e) {
    // connection/network error
} catch (BaseServerResponseException e) {
    OperationOutcome outcome = (OperationOutcome) e.getOperationOutcome();
    if (outcome != null) {
        outcome.getIssue().forEach(issue ->
            log.error("FHIR error: {} - {}", issue.getSeverity(), issue.getDiagnostics()));
    }
}
```

## Additional Resources

- For Kanta profiles and extensions detail: [kanta-reference.md](kanta-reference.md)
- For HAPI FHIR client examples: [hapi-fhir-examples.md](hapi-fhir-examples.md)
- FHIR R4 spec: https://www.hl7.org/fhir/overview.html
- HAPI FHIR docs: https://hapifhir.io/hapi-fhir/docs/client/generic_client.html
- Kanta medication list: https://simplifier.net/guide/finnish-kanta-medication-list-r4?version=current
