---
name: generate
description: Generate synthetic FHIR-compliant patient data with diagnosis-driven realistic patterns
allowed-tools:
  - Read
  - Write
  - AskUserQuestion
  - Bash
argument-hint: "[--patients N] [--diagnosis CONDITION] [--age-range MIN-MAX] [--gender male|female|other] [--timespan Ny]"
---

# Generate Synthetic Patient Data

Generate FHIR R4-compliant synthetic healthcare data with realistic longitudinal patient histories driven by diagnoses.

## Overview

This command creates synthetic patient records for EHR testing without exposing real patient information. Data generation is diagnosis-driven, creating clinically appropriate patterns including encounters, observations, medications, procedures, and other FHIR resources based on the specified conditions.

## How to Use This Command

The command works interactively, gathering required information through questions if not provided as arguments.

### Interactive Mode (Recommended)

Simply run:
```
/generate
```

Claude will ask:
1. How many patients to generate
2. Primary diagnosis (natural language or ICD-10 code)
3. Secondary diagnoses (optional)
4. Age range for patients
5. Gender distribution
6. Ethnicity (optional)
7. Time span for patient history
8. Whether to export immediately after generation

### Command-Line Arguments Mode

Provide parameters upfront:
```
/generate --patients 10 --diagnosis "Type 2 Diabetes" --age-range 45-65 --gender mixed --timespan 3y
```

**Available Arguments**:
- `--patients N`: Number of patients to generate (default: 1)
- `--diagnosis "CONDITION"`: Primary diagnosis (natural language or ICD-10 code like "E11.9")
- `--secondary "COND1,COND2"`: Comma-separated secondary diagnoses
- `--age-range MIN-MAX`: Age range in years (e.g., "45-65")
- `--gender male|female|other|mixed`: Gender distribution
- `--ethnicity "VALUE"`: Ethnicity (optional)
- `--timespan Ny`: Years of patient history (e.g., "3y" for 3 years)
- `--mode local|api`: Generation mode (default: local, API integration future)

## Instructions for Claude

When executing this command:

### Step 1: Gather Parameters

If arguments not provided, use AskUserQuestion to interactively gather:

1. **Number of patients** (default: 1)
   - Ask: "How many patients should I generate?"
   - Valid range: 1-100

2. **Primary diagnosis**
   - Ask: "What is the primary diagnosis? (You can use natural language like 'Type 2 Diabetes' or ICD-10 codes like 'E11.9')"
   - Accept both natural language and ICD-10 codes
   - Use the FHIR healthcare data skill to map natural language to ICD-10

3. **Secondary diagnoses** (optional)
   - Ask: "Any secondary diagnoses? (Optional, comma-separated)"
   - Comorbidities are common and add realism

4. **Demographics**:
   - **Age range**: "What age range? (e.g., 45-65)"
   - **Gender**: "Gender distribution? (male, female, other, or mixed)"
   - **Ethnicity** (optional): "Any specific ethnicity? (Optional)"

5. **Time span**:
   - Ask: "How many years of patient history? (e.g., 1, 3, 5)"
   - Default: 1 year

6. **Generation mode**:
   - Currently only "local" is available
   - Future: API mode will integrate with bacclabs.io

### Step 2: Load FHIR Healthcare Data Skill

The FHIR healthcare data skill provides:
- ICD-10 code mappings (`references/icd10-mappings.md`)
- Clinical patterns for diagnoses (`references/clinical-patterns.md`)
- FHIR resource examples (`examples/*.json`)
- FHIR specifications (`references/fhir-specifications.md`)

Use this skill to generate realistic, clinically accurate patient data.

### Step 3: Generate Patient Data (Local Mode)

For each patient:

1. **Create Patient resource**:
   - Use `examples/patient.json` as template
   - Generate random but realistic demographics:
     - Name (diverse names matching ethnicity if specified)
     - Birth date (based on age range)
     - Gender (based on distribution)
     - Address (realistic US address)
     - Contact information (realistic phone/email)
     - Identifiers (MRN, SSN - synthetic values)

2. **Map diagnosis to ICD-10**:
   - If natural language provided, use `references/icd10-mappings.md`
   - Create Condition resource using `examples/condition.json`

3. **Load clinical pattern**:
   - Consult `references/clinical-patterns.md` for diagnosis-specific workflows
   - Determine encounter frequency, typical observations, medications, procedures

4. **Build longitudinal timeline**:
   - **Initial presentation**: First encounter with diagnosis
   - **Diagnostic tests**: Observations confirming diagnosis
   - **Treatment initiation**: Medications, care plan
   - **Follow-up visits**: Regular encounters based on condition
   - **Progression**: Changes over time (improvement, complications)

5. **Generate FHIR resources**:
   - **Encounters**: Office visits, hospitalizations (use `examples/encounter.json`)
   - **Observations**: Labs, vitals (use `examples/observation-*.json`)
   - **Medications**: Prescriptions (use `examples/medication.json`)
   - **Procedures**: Screenings, treatments (use `examples/procedure.json`)
   - **Other resources**: Allergies, immunizations, diagnostic reports, care plans

6. **Ensure referential integrity**:
   - All resources reference Patient
   - Observations reference Encounters
   - DiagnosticReports reference Observations
   - MedicationStatements reference Conditions

7. **Validate realism**:
   - Dates progress chronologically
   - Lab values in appropriate ranges (consult `references/clinical-patterns.md`)
   - Medications appropriate for diagnosis
   - Encounter frequency matches severity

### Step 4: Store Generated Data

Store generated data in memory or temporary files:
- Create JSON Bundle containing all resources for each patient
- Organize as: one Bundle per patient
- Store in variable or write to temp directory for export

### Step 5: Present Summary

Show user:
- Number of patients generated
- Primary/secondary diagnoses
- Time span covered
- Total number of FHIR resources created (breakdown by type)
- Sample of generated data (first patient summary)

Example:
```
✓ Generated 10 patients with Type 2 Diabetes
  - Time span: 3 years per patient
  - Age range: 45-65 years
  - Gender: Mixed (5 male, 5 female)

Resources created per patient:
  - 1 Patient
  - 1 Primary Condition (Type 2 Diabetes)
  - 12 Encounters (quarterly visits)
  - 48 Observations (vitals + labs)
  - 4 Medication Statements
  - 3 Procedures (annual screenings)
  - 1 Care Plan
  - Total: 70 resources × 10 patients = 700 FHIR resources

Sample patient: John Smith (MRN: MRN789012)
  - Age: 52, Male
  - Diagnosed: 2021-03-15
  - Latest A1C: 6.8% (well-controlled)
  - Medications: Metformin 1000mg BID
```

### Step 6: Auto-Validate Generated Data (NEW in v0.2.0)

Before offering export, automatically validate the generated data for quality:

1. **Check validation settings** in `.claude/cynthia-generator.local.md`:
   ```markdown
   ## Validation Settings
   - Auto-validate after generate: yes|no (default: yes)
   ```

2. **If auto-validate enabled (default: yes)**:
   - Invoke the `fhir-data-validator` agent with the generated bundle
   - Use Task tool with subagent_type='general-purpose'
   - Pass context: "Validate this FHIR Bundle for clinical realism and referential integrity"

   **Show brief validation summary**:

   If **NO CRITICAL issues found**:
   ```
   ✓ Validation complete: No critical issues found
     (2 minor suggestions - run /validate for details)
   ```

   If **WARNINGS found**:
   ```
   ✓ Validation complete: No critical issues
     ⚠️  Found 3 warnings - run /validate to review
   ```

   If **CRITICAL issues found**:
   ```
   ⚠️  Validation found 3 critical issues
      Run /validate to see details and fix before export

      Issues detected:
      - Broken reference (Patient ID mismatch)
      - Missing required field (Condition.code)
      - Impossible value (negative age)
   ```

3. **If auto-validate disabled**:
   - Skip validation step
   - Show note:
   ```
   ℹ️  Auto-validation skipped (disabled in settings)
      Tip: Run /validate manually to check data quality
   ```

4. **If validation agent unavailable or errors**:
   - Continue without validation
   - Show note:
   ```
   ℹ️  Validation unavailable - proceeding without quality check
      Tip: Run /validate manually when ready
   ```

### Step 7: Offer Export

Ask user: "Would you like to export this data now?"

Options:
- **Yes**: Proceed to execute `/export` command automatically
- **No**: Data remains in memory for later export
- **Show details**: Display more information about a specific patient
- **Validate first**: If critical issues found, suggest running /validate before export

If user chooses export, call the export command with generated data.

**If CRITICAL issues were found**:
- Recommend fixing issues first: "I recommend running /validate and fixing critical issues before export. Export anyway?"

## Important Considerations

### Clinical Realism

- Use diagnosis-specific patterns from `references/clinical-patterns.md`
- Follow realistic encounter frequencies (monthly, quarterly, annually)
- Generate lab values that trend realistically over time
- Include appropriate complications for poorly controlled conditions
- Account for medication adherence patterns

### FHIR Compliance

- All resources must validate against FHIR R4 schema
- Use standard code systems (ICD-10, LOINC, RxNorm, CPT, SNOMED)
- Maintain referential integrity between resources
- Include required fields for each resource type

### Data Privacy

- Generate completely synthetic data (no real patient information)
- Use realistic but fake names, addresses, phone numbers
- Create synthetic MRNs and SSNs
- Ensure data cannot be traced to real individuals

### Comorbidity Support

When multiple diagnoses specified:
- Consider common comorbidity patterns (diabetes + hypertension)
- Sequence diagnoses chronologically (which came first)
- Generate overlapping care (single visit addresses multiple conditions)
- Account for medication interactions
- Increase encounter frequency appropriately

### Performance

- For large patient counts (50+), show progress indicator
- Consider batching generation to avoid memory issues
- Warn if generation will take significant time

## Settings Integration

Check for settings in `.claude/cynthia-generator.local.md`:
- Default patient count
- Default time span
- Default demographics
- API credentials (future)

If settings exist, use as defaults. Otherwise use hardcoded defaults.

## Error Handling

Handle these scenarios gracefully:
- Invalid diagnosis name (suggest similar diagnoses)
- Invalid ICD-10 code (provide correction)
- Invalid age range (request correction)
- Invalid time span (request correction)
- Missing required parameters (prompt for them)

## Examples

### Example 1: Single Diabetes Patient
```
/generate

> How many patients? 1
> Primary diagnosis? Type 2 Diabetes
> Age range? 50-60
> Gender? male
> Time span? 3 years
> Export now? yes
```

### Example 2: Multiple Patients with Comorbidities
```
/generate --patients 10 --diagnosis "E11.9" --secondary "I10,E66.9" --age-range 45-70 --timespan 5y

> Generated 10 patients with:
>   - Type 2 Diabetes (E11.9)
>   - Essential Hypertension (I10)
>   - Obesity (E66.9)
> Ready to export.
```

### Example 3: COPD Patients
```
/generate --patients 5 --diagnosis "Chronic Obstructive Pulmonary Disease" --age-range 55-75 --gender mixed --timespan 3y
```

## Tips

- Start with 1-2 patients to verify data quality before generating large batches
- Use ICD-10 codes for precise diagnosis specification
- Include comorbidities for more realistic populations
- Longer time spans (3-5 years) create richer longitudinal histories
- Review generated data before export to ensure it meets your needs

## Future Enhancements

When API mode is available:
- `--mode api` will delegate generation to bacclabs.io API
- Faster generation for large patient counts
- Access to more sophisticated pattern generation
- Consistent with bacclabs.io's generation algorithms

---

Remember: You are generating instructions FOR Claude, not TO the user. Write in imperative form describing what Claude should do when executing this command.
