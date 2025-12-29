# FHIR R4 Resource Specifications

Quick reference for FHIR R4 resource structure, required fields, and common value sets. For complete specifications, refer to https://hl7.org/fhir/R4/

## Common Elements

All FHIR resources share these elements:

```json
{
  "resourceType": "ResourceName",
  "id": "unique-identifier",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2024-01-15T10:30:00Z"
  }
}
```

### Reference Format

Resources reference each other using:
```json
{
  "reference": "Patient/patient-123",
  "display": "John Doe"
}
```

### Coding Systems

Use standard code systems:
- **ICD-10**: Diagnoses (http://hl7.org/fhir/sid/icd-10-cm)
- **LOINC**: Lab observations (http://loinc.org)
- **RxNorm**: Medications (http://www.nlm.nih.gov/research/umls/rxnorm)
- **CPT**: Procedures (http://www.ama-assn.org/go/cpt)
- **SNOMED CT**: Clinical terms (http://snomed.info/sct)

---

## Patient Resource

**Purpose**: Demographics and patient identifiers

**Required Fields**:
- `resourceType`: "Patient"
- `identifier`: At least one identifier (MRN, SSN)
- `name`: At least one name

**Key Fields**:
```json
{
  "resourceType": "Patient",
  "id": "patient-123",
  "identifier": [{
    "system": "http://hospital.org/mrn",
    "value": "MRN12345"
  }],
  "name": [{
    "use": "official",
    "family": "Doe",
    "given": ["John", "Robert"]
  }],
  "gender": "male",
  "birthDate": "1965-08-15",
  "address": [{
    "line": ["123 Main St"],
    "city": "Boston",
    "state": "MA",
    "postalCode": "02101",
    "country": "USA"
  }],
  "telecom": [{
    "system": "phone",
    "value": "617-555-1234",
    "use": "home"
  }]
}
```

**Gender Values**: male, female, other, unknown

---

## Encounter Resource

**Purpose**: Healthcare visit or episode of care

**Required Fields**:
- `resourceType`: "Encounter"
- `status`: Current status of encounter
- `class`: Classification of encounter
- `subject`: Reference to Patient

**Key Fields**:
```json
{
  "resourceType": "Encounter",
  "id": "encounter-456",
  "status": "finished",
  "class": {
    "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
    "code": "AMB",
    "display": "ambulatory"
  },
  "type": [{
    "coding": [{
      "system": "http://snomed.info/sct",
      "code": "390906007",
      "display": "Follow-up encounter"
    }]
  }],
  "subject": {
    "reference": "Patient/patient-123"
  },
  "period": {
    "start": "2024-01-15T10:00:00Z",
    "end": "2024-01-15T10:45:00Z"
  },
  "reasonReference": [{
    "reference": "Condition/condition-789"
  }]
}
```

**Status Values**: planned, arrived, triaged, in-progress, onleave, finished, cancelled

**Class Codes**:
- AMB: Ambulatory (outpatient)
- EMER: Emergency
- IMP: Inpatient
- ACUTE: Acute inpatient
- NONAC: Non-acute inpatient

---

## Observation Resource

**Purpose**: Measurements, lab results, vital signs

**Required Fields**:
- `resourceType`: "Observation"
- `status`: Status of observation
- `code`: What was observed (LOINC code)
- `subject`: Reference to Patient

**Vital Signs Example**:
```json
{
  "resourceType": "Observation",
  "id": "obs-vitals-001",
  "status": "final",
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/observation-category",
      "code": "vital-signs"
    }]
  }],
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "85354-9",
      "display": "Blood pressure panel"
    }]
  },
  "subject": {
    "reference": "Patient/patient-123"
  },
  "effectiveDateTime": "2024-01-15T10:15:00Z",
  "component": [{
    "code": {
      "coding": [{
        "system": "http://loinc.org",
        "code": "8480-6",
        "display": "Systolic blood pressure"
      }]
    },
    "valueQuantity": {
      "value": 128,
      "unit": "mmHg",
      "system": "http://unitsofmeasure.org",
      "code": "mm[Hg]"
    }
  }, {
    "code": {
      "coding": [{
        "system": "http://loinc.org",
        "code": "8462-4",
        "display": "Diastolic blood pressure"
      }]
    },
    "valueQuantity": {
      "value": 82,
      "unit": "mmHg",
      "system": "http://unitsofmeasure.org",
      "code": "mm[Hg]"
    }
  }]
}
```

**Status Values**: registered, preliminary, final, amended, cancelled

**Common LOINC Codes**:
- 85354-9: Blood pressure panel
- 8867-4: Heart rate
- 8310-5: Body temperature
- 2708-6: Oxygen saturation
- 4548-4: Hemoglobin A1C
- 2339-0: Glucose
- 2093-3: Total cholesterol

---

## Condition Resource

**Purpose**: Diagnoses, problems, health conditions

**Required Fields**:
- `resourceType`: "Condition"
- `code`: Diagnosis code (ICD-10)
- `subject`: Reference to Patient

**Key Fields**:
```json
{
  "resourceType": "Condition",
  "id": "condition-789",
  "clinicalStatus": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/condition-clinical",
      "code": "active"
    }]
  },
  "verificationStatus": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/condition-ver-status",
      "code": "confirmed"
    }]
  },
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/condition-category",
      "code": "encounter-diagnosis"
    }]
  }],
  "code": {
    "coding": [{
      "system": "http://hl7.org/fhir/sid/icd-10-cm",
      "code": "E11.9",
      "display": "Type 2 diabetes mellitus without complications"
    }]
  },
  "subject": {
    "reference": "Patient/patient-123"
  },
  "onsetDateTime": "2023-06-10",
  "recordedDate": "2023-06-10"
}
```

**Clinical Status**: active, recurrence, relapse, inactive, remission, resolved

**Verification Status**: unconfirmed, provisional, differential, confirmed, refuted

---

## Medication Resource / MedicationStatement

**Purpose**: Medication information and patient medication records

**MedicationStatement** (records that patient is/was taking medication):
```json
{
  "resourceType": "MedicationStatement",
  "id": "med-statement-001",
  "status": "active",
  "medicationCodeableConcept": {
    "coding": [{
      "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
      "code": "860975",
      "display": "Metformin 500 MG Oral Tablet"
    }]
  },
  "subject": {
    "reference": "Patient/patient-123"
  },
  "effectivePeriod": {
    "start": "2023-06-10"
  },
  "dosage": [{
    "text": "500 mg orally twice daily",
    "timing": {
      "repeat": {
        "frequency": 2,
        "period": 1,
        "periodUnit": "d"
      }
    },
    "doseAndRate": [{
      "doseQuantity": {
        "value": 500,
        "unit": "mg",
        "system": "http://unitsofmeasure.org",
        "code": "mg"
      }
    }]
  }]
}
```

**Status Values**: active, completed, entered-in-error, intended, stopped, on-hold

---

## Procedure Resource

**Purpose**: Surgical or diagnostic procedures

**Required Fields**:
- `resourceType`: "Procedure"
- `status`: Status of procedure
- `code`: Type of procedure (CPT/SNOMED)
- `subject`: Reference to Patient

**Key Fields**:
```json
{
  "resourceType": "Procedure",
  "id": "procedure-001",
  "status": "completed",
  "code": {
    "coding": [{
      "system": "http://snomed.info/sct",
      "code": "252160004",
      "display": "Complete physical examination"
    }]
  },
  "subject": {
    "reference": "Patient/patient-123"
  },
  "performedDateTime": "2024-01-15T10:30:00Z",
  "encounter": {
    "reference": "Encounter/encounter-456"
  }
}
```

**Status Values**: preparation, in-progress, not-done, on-hold, stopped, completed

---

## AllergyIntolerance Resource

**Purpose**: Allergies and adverse reactions

**Required Fields**:
- `resourceType`: "AllergyIntolerance"
- `patient`: Reference to Patient
- `code`: Allergen

**Key Fields**:
```json
{
  "resourceType": "AllergyIntolerance",
  "id": "allergy-001",
  "clinicalStatus": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/allergyintolerance-clinical",
      "code": "active"
    }]
  },
  "verificationStatus": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/allergyintolerance-verification",
      "code": "confirmed"
    }]
  },
  "type": "allergy",
  "category": ["medication"],
  "criticality": "high",
  "code": {
    "coding": [{
      "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
      "code": "7980",
      "display": "Penicillin"
    }]
  },
  "patient": {
    "reference": "Patient/patient-123"
  },
  "reaction": [{
    "substance": {
      "coding": [{
        "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
        "code": "7980",
        "display": "Penicillin"
      }]
    },
    "manifestation": [{
      "coding": [{
        "system": "http://snomed.info/sct",
        "code": "247472004",
        "display": "Urticaria"
      }]
    }],
    "severity": "moderate"
  }]
}
```

**Type**: allergy, intolerance
**Category**: food, medication, environment, biologic
**Criticality**: low, high, unable-to-assess

---

## Immunization Resource

**Purpose**: Vaccination records

**Required Fields**:
- `resourceType`: "Immunization"
- `status`: Status of immunization
- `vaccineCode`: Type of vaccine
- `patient`: Reference to Patient
- `occurrenceDateTime`: When administered

**Key Fields**:
```json
{
  "resourceType": "Immunization",
  "id": "immunization-001",
  "status": "completed",
  "vaccineCode": {
    "coding": [{
      "system": "http://hl7.org/fhir/sid/cvx",
      "code": "140",
      "display": "Influenza, seasonal, injectable, preservative free"
    }]
  },
  "patient": {
    "reference": "Patient/patient-123"
  },
  "occurrenceDateTime": "2024-10-15",
  "primarySource": true
}
```

**Status Values**: completed, entered-in-error, not-done

---

## DiagnosticReport Resource

**Purpose**: Clinical reports from labs, imaging, pathology

**Required Fields**:
- `resourceType`: "DiagnosticReport"
- `status`: Status of report
- `code`: Type of diagnostic report
- `subject`: Reference to Patient

**Key Fields**:
```json
{
  "resourceType": "DiagnosticReport",
  "id": "diagnostic-report-001",
  "status": "final",
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v2-0074",
      "code": "LAB"
    }]
  }],
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "58410-2",
      "display": "Complete blood count (hemogram) panel"
    }]
  },
  "subject": {
    "reference": "Patient/patient-123"
  },
  "effectiveDateTime": "2024-01-15T08:00:00Z",
  "issued": "2024-01-15T14:30:00Z",
  "result": [{
    "reference": "Observation/obs-hemoglobin"
  }, {
    "reference": "Observation/obs-wbc"
  }]
}
```

**Status Values**: registered, partial, preliminary, final, amended, corrected, cancelled

---

## CarePlan Resource

**Purpose**: Treatment and care plans

**Required Fields**:
- `resourceType`: "CarePlan"
- `status`: Status of care plan
- `intent`: Intent of care plan
- `subject`: Reference to Patient

**Key Fields**:
```json
{
  "resourceType": "CarePlan",
  "id": "careplan-001",
  "status": "active",
  "intent": "plan",
  "title": "Diabetes Management Plan",
  "description": "Comprehensive care plan for Type 2 Diabetes management",
  "subject": {
    "reference": "Patient/patient-123"
  },
  "period": {
    "start": "2023-06-10"
  },
  "addresses": [{
    "reference": "Condition/condition-789"
  }],
  "activity": [{
    "detail": {
      "kind": "MedicationRequest",
      "code": {
        "text": "Metformin 500mg BID"
      },
      "status": "in-progress"
    }
  }, {
    "detail": {
      "kind": "ServiceRequest",
      "code": {
        "text": "Blood glucose self-monitoring"
      },
      "status": "in-progress"
    }
  }]
}
```

**Status Values**: draft, active, on-hold, revoked, completed
**Intent Values**: proposal, plan, order, option

---

## Data Validation

### Required Element Checks

Before exporting, validate:
- All required fields present
- Resource references point to valid resources
- Dates are in ISO 8601 format
- Code systems use standard URIs
- Status values from allowed value sets

### Common Validation Errors

**Missing Required Fields**:
- Patient without identifier
- Observation without code
- Condition without subject reference

**Invalid References**:
- Reference to non-existent patient
- Circular references between resources

**Invalid Codes**:
- ICD-10 code in medication field (should be RxNorm)
- Non-standard code system URIs

### FHIR Validation Tools

For production validation:
- Official FHIR Validator: https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator
- Online validator: https://validator.fhir.org

---

## Bundle Resource (for Multi-Resource Export)

When exporting multiple resources, wrap in a Bundle:

```json
{
  "resourceType": "Bundle",
  "type": "collection",
  "entry": [{
    "resource": {
      "resourceType": "Patient",
      "id": "patient-123"
      // ... patient details
    }
  }, {
    "resource": {
      "resourceType": "Encounter",
      "id": "encounter-456"
      // ... encounter details
    }
  }]
}
```

**Bundle Types**:
- document: Document bundle
- message: Message bundle
- transaction: Transaction (create/update/delete)
- collection: Collection of resources
- batch: Batch processing

---

## Best Practices

1. **Use standard code systems**: ICD-10 for diagnoses, RxNorm for medications, LOINC for labs
2. **Maintain referential integrity**: Ensure all resource references are valid
3. **Include metadata**: Always populate meta.lastUpdated
4. **Use appropriate status values**: Match status to actual resource state
5. **Provide display values**: Include human-readable text with coded values
6. **Follow chronology**: Ensure dates progress logically
7. **Bundle related resources**: Export as Bundle for complete patient records

Consult https://hl7.org/fhir/R4/ for complete specification details.
