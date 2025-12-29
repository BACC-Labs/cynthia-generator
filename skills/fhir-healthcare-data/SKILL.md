---
name: FHIR Healthcare Data Generation
description: This skill should be used when the user asks to "generate synthetic health data", "create FHIR patient data", "generate patient records", "create EHR test data", mentions diagnoses like "diabetes patient", "hypertension", uses ICD-10 codes like "E11.9", or asks about "longitudinal patient history", "FHIR resources", "healthcare data patterns", or "realistic medical records". Provides comprehensive guidance for generating FHIR R4-compliant synthetic healthcare data with diagnosis-driven realistic patterns.
version: 0.1.0
---

# FHIR Healthcare Data Generation

## Overview

Generate synthetic healthcare data following FHIR (Fast Healthcare Interoperability Resources) R4 specifications. This skill provides knowledge for creating realistic longitudinal patient histories driven by primary and secondary diagnoses, ensuring data patterns match real-world clinical workflows while maintaining HIPAA compliance through synthetic generation.

FHIR is the healthcare industry standard for exchanging electronic health information. All generated data must conform to FHIR R4 resource specifications to ensure compatibility with EHR systems, FHIR servers, and healthcare applications.

## When to Use This Skill

Use this skill when:
- Generating synthetic patient data for EHR testing
- Creating FHIR-compliant healthcare records
- Building longitudinal patient histories based on diagnoses
- Simulating realistic clinical patterns and workflows
- Producing test data that avoids HIPAA violations
- Validating healthcare applications with realistic data

## Core FHIR Resources

The following FHIR resources form the foundation of comprehensive patient records:

### Patient
Demographic information and patient identifiers. Every healthcare record starts with a Patient resource containing name, gender, birth date, address, contact information, and identifiers (MRN, SSN).

### Encounter
Healthcare visits including office visits, hospital admissions, emergency department visits, and telehealth appointments. Encounters link to the reason for visit (Condition), observations made, and procedures performed.

### Observation
Clinical measurements and lab results including vital signs (blood pressure, heart rate, temperature, oxygen saturation), laboratory values (A1C, glucose, lipid panels, kidney function), and clinical assessments.

### Condition
Diagnoses and health problems including primary diagnoses, secondary conditions, complications, and resolved conditions. Each Condition includes ICD-10 codes, onset date, clinical status, and verification status.

### Medication
Prescriptions and medication administration including medication statements, prescription details, dosage instructions, refill patterns, and adherence records.

### Procedure
Surgeries, treatments, and medical procedures including surgical interventions, diagnostic procedures, therapeutic procedures, and preventive services. Each includes procedure codes (CPT, SNOMED), date performed, and outcomes.

### AllergyIntolerance
Patient allergies and adverse reactions including drug allergies, food allergies, environmental allergies, severity levels, and reaction types.

### Immunization
Vaccination records including vaccine types, administration dates, lot numbers, and immunization schedules following CDC guidelines.

### DiagnosticReport
Lab reports and imaging results including complete blood counts, metabolic panels, imaging studies (X-ray, CT, MRI), and pathology reports.

### CarePlan
Treatment and care plans including goals, interventions, medications, lifestyle modifications, and monitoring schedules.

## Diagnosis-Driven Data Generation

Generate realistic patient histories by starting with a primary diagnosis and building clinically appropriate patterns around it.

### ICD-10 Code Support

Accept diagnoses in two formats:
- **Natural language**: "Type 2 Diabetes", "Hypertension", "Chronic Kidney Disease"
- **ICD-10 codes**: "E11.9", "I10", "N18.3"

Map natural language to ICD-10 codes using `references/icd10-mappings.md`. Common examples:
- Type 2 Diabetes → E11.9 (without complications) or E11.65 (with hyperglycemia)
- Essential Hypertension → I10
- Coronary Artery Disease → I25.10
- Chronic Obstructive Pulmonary Disease → J44.9
- Obesity → E66.9

### Clinical Pattern Generation

For each diagnosis, generate appropriate associated resources following realistic clinical workflows. Consult `references/clinical-patterns.md` for detailed diagnosis-specific patterns.

**Example: Type 2 Diabetes (E11.9)**

Typical pattern over 3 years:
1. **Initial diagnosis** (Year 0):
   - Encounter: Office visit with symptoms (polyuria, polydipsia, fatigue)
   - Observations: Elevated fasting glucose (>126 mg/dL), elevated A1C (>6.5%)
   - Condition: Type 2 Diabetes diagnosis (ICD-10: E11.9)
   - Medication: Metformin 500mg twice daily
   - CarePlan: Blood glucose monitoring, dietary modifications, exercise

2. **Follow-up visits** (Every 3 months):
   - Encounter: Diabetes management visit
   - Observations: A1C tests (target <7%), fasting glucose, weight, blood pressure
   - Medication: Dosage adjustments based on A1C trends

3. **Progression** (Years 1-3):
   - Observations: A1C may rise if uncontrolled (7-9%), or improve if well-managed (<7%)
   - Medications: Add second agent if A1C >7% (e.g., GLP-1 agonist, DPP-4 inhibitor)
   - Procedures: Annual diabetic retinopathy screening, foot exams
   - Conditions: Possible complications (neuropathy, nephropathy) if poorly controlled

**Example: Hypertension (I10)**

Typical pattern:
1. **Initial diagnosis**:
   - Encounter: Annual physical or symptomatic visit
   - Observations: Elevated BP readings (>140/90 mmHg on multiple occasions)
   - Condition: Essential Hypertension (ICD-10: I10)
   - Medication: ACE inhibitor or thiazide diuretic

2. **Monitoring** (Monthly initially, then quarterly):
   - Observations: Blood pressure measurements
   - Medication: Titrate dosage or add second agent if BP not controlled

3. **Long-term management**:
   - Observations: Regular BP monitoring, kidney function tests, electrolytes
   - CarePlan: Lifestyle modifications (low sodium diet, exercise)

### Longitudinal History Construction

Build patient histories chronologically with realistic timing:

1. **Establish baseline**: Start with patient demographics and any pre-existing conditions
2. **Initial presentation**: First encounter with symptoms leading to diagnosis
3. **Diagnosis confirmation**: Tests confirming the diagnosis
4. **Treatment initiation**: First medications and care plan
5. **Follow-up pattern**: Regular visits based on condition severity
6. **Progression modeling**: Changes over time (improvement, deterioration, complications)
7. **Related conditions**: Secondary diagnoses that commonly co-occur

Consult `references/clinical-patterns.md` for timing guidelines by diagnosis type.

### Multi-Diagnosis Patients

When generating patients with multiple conditions:
1. **Consider comorbidity patterns**: Diabetes and hypertension commonly co-occur
2. **Sequence appropriately**: Establish which diagnosis came first chronologically
3. **Generate overlapping care**: Single encounters may address multiple conditions
4. **Account for interactions**: Medications for one condition may affect another
5. **Increase encounter frequency**: Multiple conditions mean more frequent visits

Common comorbidity clusters:
- Diabetes + Hypertension + Obesity (metabolic syndrome)
- COPD + Coronary Artery Disease (smoking-related)
- Chronic Kidney Disease + Hypertension + Diabetes
- Congestive Heart Failure + Coronary Artery Disease

## Data Generation Modes

### Local Generation

Generate data using hybrid template-algorithm approach:
1. **Load templates**: Use FHIR JSON templates from `examples/` directory
2. **Populate demographics**: Randomize patient identifiers, names, addresses within specified parameters
3. **Apply diagnosis patterns**: Use clinical patterns from `references/clinical-patterns.md`
4. **Generate timeline**: Create chronological sequence of encounters and observations
5. **Fill values**: Populate lab values, vital signs with realistic ranges and trends
6. **Link resources**: Ensure proper FHIR references between resources (Encounter → Condition, DiagnosticReport → Observation)
7. **Validate structure**: Confirm all resources conform to FHIR R4 schema

**Realism level (MVP)**: Basic realism ensures:
- Dates are chronological and realistic (no future dates, proper spacing)
- Lab values within medically plausible ranges
- Medications appropriate for diagnosed conditions
- Encounter frequency matches condition severity
- Patient age appropriate for diagnosis

### API Generation (Future)

When API integration is available, delegate generation to external service while maintaining FHIR compliance validation locally.

## FHIR Resource Examples

Complete example FHIR resources available in `examples/` directory:

- **`examples/patient.json`** - Patient resource with complete demographics
- **`examples/encounter.json`** - Office visit encounter
- **`examples/observation-vitals.json`** - Vital signs observation
- **`examples/observation-labs.json`** - Laboratory results
- **`examples/condition.json`** - Diagnosis/condition
- **`examples/medication.json`** - Medication statement
- **`examples/procedure.json`** - Surgical procedure
- **`examples/allergy.json`** - Allergy intolerance
- **`examples/immunization.json`** - Vaccination record
- **`examples/diagnostic-report.json`** - Lab report
- **`examples/care-plan.json`** - Treatment care plan

Use these templates as starting points, modifying values to match specific patient scenarios.

## Realistic Value Ranges

Generate clinically realistic values for observations:

**Vital Signs** (Normal adult ranges):
- Blood Pressure: 90-120 / 60-80 mmHg (higher for hypertension)
- Heart Rate: 60-100 bpm
- Temperature: 97.8-99.1°F (36.5-37.3°C)
- Respiratory Rate: 12-20 breaths/min
- Oxygen Saturation: 95-100%
- BMI: 18.5-24.9 (normal), 25-29.9 (overweight), 30+ (obese)

**Common Lab Values**:
- Fasting Glucose: 70-100 mg/dL (normal), 100-125 (prediabetic), 126+ (diabetic)
- A1C: <5.7% (normal), 5.7-6.4% (prediabetic), ≥6.5% (diabetic)
- Total Cholesterol: <200 mg/dL (desirable)
- LDL Cholesterol: <100 mg/dL (optimal)
- HDL Cholesterol: >40 mg/dL men, >50 mg/dL women
- Creatinine: 0.7-1.3 mg/dL
- eGFR: >60 mL/min/1.73m² (normal kidney function)

Adjust values based on diagnosis and disease progression. Consult `references/clinical-patterns.md` for diagnosis-specific abnormal ranges.

## Data Quality Standards

Ensure generated data meets these quality criteria:

1. **FHIR Compliance**: All resources validate against FHIR R4 schema
2. **Clinical Accuracy**: Diagnoses, medications, and procedures are appropriate combinations
3. **Chronological Consistency**: Dates progress logically, no time travel
4. **Referential Integrity**: Resource references point to valid resources
5. **Coding Systems**: Use standard code systems (ICD-10, CPT, LOINC, RxNorm, SNOMED)
6. **Demographic Diversity**: Generate varied populations unless user specifies otherwise
7. **Privacy Compliance**: No real patient data, all synthesized values

## Additional Resources

### Reference Files

For detailed clinical patterns and specifications:
- **`references/clinical-patterns.md`** - Diagnosis-specific clinical patterns, encounter frequencies, medication protocols, progression timelines
- **`references/icd10-mappings.md`** - Common diagnoses mapped to ICD-10 codes
- **`references/fhir-specifications.md`** - FHIR R4 resource specifications, required fields, value sets

### Example Files

Complete working FHIR JSON examples in `examples/`:
- All major FHIR resource types with realistic values
- Properly structured and validated JSON
- Reference-linked resources demonstrating relationships

Use examples as templates, modifying values for specific scenarios while maintaining FHIR structure.

## Implementation Guidelines

When generating synthetic healthcare data:

1. **Start with patient context**: Gather required parameters (age, gender, diagnosis, timespan)
2. **Map diagnosis**: Convert natural language to ICD-10 code using `references/icd10-mappings.md`
3. **Load clinical pattern**: Consult `references/clinical-patterns.md` for diagnosis-specific workflow
4. **Create Patient resource**: Generate demographics using `examples/patient.json` as template
5. **Build timeline**: Create chronological sequence of Encounters based on pattern
6. **Add clinical data**: Generate Observations, Conditions, Medications per encounter
7. **Include supporting data**: Add Procedures, Immunizations, Allergies as appropriate
8. **Link resources**: Ensure proper FHIR references between related resources
9. **Validate output**: Confirm FHIR compliance and clinical realism
10. **Prepare for export**: Format according to user's export destination requirements

Focus on creating clinically coherent narratives that tell a realistic patient story over time.
