---
name: validate
description: Validate synthetic FHIR patient data for clinical realism, referential integrity, and FHIR R4 compliance
allowed-tools:
  - Read
  - AskUserQuestion
  - Task
argument-hint: "[--file PATH] [--severity all|critical|warnings]"
---

# Validate Synthetic Patient Data

Validate generated FHIR-compliant synthetic healthcare data for clinical accuracy, referential integrity, and structural compliance.

## Overview

This command analyzes FHIR Bundles to check:
- **Clinical Realism**: Lab values, medication dosages, disease progression patterns match medical standards
- **Referential Integrity**: All resource references (Patient → Condition → Encounter → Observation) are valid
- **FHIR Compliance**: Resources conform to FHIR R4 specifications (required fields, code systems, dates)

## How to Use This Command

### Interactive Mode (Recommended)

Simply run:
```
/validate
```

Claude will ask:
1. Path to FHIR Bundle JSON file to validate
2. Severity level to report (all issues, critical only, or critical+warnings)

### Command-Line Arguments Mode

Provide parameters upfront:
```
/validate --file ./synthetic-patient-001.json
/validate --file ./test-data.json --severity critical
```

**Available Arguments**:
- `--file PATH`: Path to FHIR Bundle JSON file
- `--severity all|warnings|critical`: Which severity levels to report (default: all)

## Instructions for Claude

When executing this command:

### Step 1: Gather Parameters

If arguments not provided, use AskUserQuestion to gather:

1. **File path**:
   - Ask: "Which FHIR Bundle file should I validate?"
   - Default suggestion: Look for recently generated files in current directory
   - Check if file exists using Read tool
   - If not found: Ask for corrected path or list available .json files

2. **Severity level** (optional):
   - Ask: "Which severity levels should I report?"
   - Options: "all" (CRITICAL + WARNING + INFO), "warnings" (CRITICAL + WARNING), "critical" (CRITICAL only)
   - Default: "all"
   - Explain: "all = comprehensive report, warnings = actionable issues only, critical = must-fix errors only"

### Step 2: Verify File Exists

Use Read tool to check if file exists:
```
Read the first few lines to verify it's valid JSON and a FHIR Bundle
```

If file not found or not accessible:
```
✗ File not found: [path]

Please check the file path and try again.
Tip: Use absolute paths or paths relative to current directory.
```

If file is not valid JSON:
```
✗ Invalid JSON format

Error: [parse error details]
Cannot validate - please fix JSON syntax first
```

If file is not a FHIR Bundle:
```
✗ Not a FHIR Bundle

Found resourceType: "[actual type]"
Expected: "Bundle"

Tip: This command validates Bundles containing multiple resources.
For individual resources, wrap them in a Bundle first.
```

### Step 3: Invoke FHIR Data Validator Agent

Trigger the fhir-data-validator agent with the file path:

```
Use Task tool with subagent_type='general-purpose' to invoke the fhir-data-validator agent

Pass the file path as context:
"Please validate the FHIR Bundle at [file path] for clinical realism, referential integrity, and FHIR compliance."
```

The agent will:
1. Load and parse the FHIR Bundle
2. Build resource index
3. Run clinical realism checks (lab values, medications, progression patterns)
4. Run referential integrity checks (verify all IDs exist, references valid)
5. Run structural checks (required fields, code systems, dates)
6. Generate comprehensive validation report with severity levels

### Step 4: Receive and Process Agent Report

The agent returns a structured report with:
- **Summary**: Total resources, issue counts by severity
- **CRITICAL issues**: Must-fix errors (broken references, missing required fields, impossible values)
- **WARNING issues**: Unusual patterns (extreme lab values, odd medication dosages)
- **INFO suggestions**: Enhancement opportunities (missing optional screenings)

### Step 5: Filter Report by Severity

Apply user's severity filter:

**If severity = "all"**:
- Display complete report (CRITICAL + WARNING + INFO)

**If severity = "warnings"**:
- Display CRITICAL and WARNING only
- Hide INFO suggestions
- Note at end: "([count] info suggestions hidden - run with --severity all to see them)"

**If severity = "critical"**:
- Display CRITICAL only
- Hide WARNING and INFO
- Note at end: "([count] warnings hidden - run with --severity all to see them)"

### Step 6: Present Filtered Report

Display the validation report in a clear, formatted manner:

**Example CRITICAL Issues (FAIL)**:
```
❌ VALIDATION FAILED

## Summary
- Total resources: 70
- Critical issues: 3
- Warnings: 5
- Info: 2

## CRITICAL Issues (3)

[CRITICAL] Referential Integrity: Missing patient reference
  Resource: Observation/obs-hba1c-001
  Details: subject.reference points to Patient/patient-999 which does not exist in bundle
  Recommendation: Update reference to valid patient ID (Patient/patient-syn-001) or add missing Patient resource

[CRITICAL] Structural: Missing required field
  Resource: Condition/cond-diabetes-001
  Details: Required field 'code' is missing - diagnosis must have ICD-10 code
  Recommendation: Add code field with ICD-10 coding (e.g., E11.9 for Type 2 Diabetes)

[CRITICAL] Clinical Realism: Impossible value
  Resource: Observation/obs-bp-002
  Details: Blood pressure value of -50/30 mmHg is impossible
  Recommendation: Correct to valid range (e.g., 120/80 mmHg for normal, 140/90+ for hypertension)

---

❌ Recommendation: Fix critical issues before using this data.
   Critical issues indicate invalid FHIR structure or medically impossible values.
```

**Example WARNINGS Only (PASS WITH WARNINGS)**:
```
✓ VALIDATION PASSED (with warnings)

## Summary
- Total resources: 70
- Critical issues: 0
- Warnings: 2
- Info: 1

## WARNING Issues (2)

[WARNING] Clinical Realism: Unusually high A1C
  Resource: Observation/obs-hba1c-001
  Details: A1C value of 13.2% is extremely high (typical range 6.5-9.0% at diagnosis)
  Recommendation: Value is possible but rare. Consider reducing to 8-9% for more typical presentation.

[WARNING] Clinical Realism: Missing first-line medication
  Resource: Patient/patient-syn-001 (Type 2 Diabetes)
  Details: Patient not on metformin or any oral glucose-lowering medication
  Recommendation: Add metformin 500mg BID as first-line treatment for Type 2 Diabetes

---

⚠️  Data is usable but consider addressing warnings for better clinical realism.
   Warnings indicate unusual but possible patterns.
```

**Example PASS (No Issues or INFO Only)**:
```
✅ VALIDATION PASSED

## Summary
- Total resources: 70
- Critical issues: 0
- Warnings: 0
- Info: 2

## INFO Suggestions (2)

[INFO] Clinical Realism: Consider adding annual screening
  Resource: Patient/patient-syn-001 (Type 2 Diabetes, 3 years history)
  Details: Diabetic patient missing annual dilated eye exam for retinopathy screening
  Recommendation: Add Procedure resource for diabetic retinopathy screening (typically annual)

[INFO] Clinical Realism: Opportunity to add immunization
  Resource: Patient/patient-syn-001 (Age 60)
  Details: Patient age-eligible for pneumonia and flu vaccinations
  Recommendation: Consider adding Immunization resources for seasonal flu and pneumococcal vaccines

---

✓ Data quality is excellent and ready for export!
  INFO suggestions are optional enhancements, not required.
```

### Step 7: Offer Next Steps

Based on validation results, suggest appropriate next actions:

**If CRITICAL issues found**:
```
Would you like me to:
1. Show you the specific resources that need fixing (I can read the file with line numbers)
2. Explain how to fix each critical issue in detail
3. Skip for now and you'll fix manually

What would you prefer?
```

**If WARNINGS only**:
```
Data passed validation with warnings. Would you like to:
1. Export the data as-is (usable but has warnings)
2. Review warnings in detail and consider improvements
3. Show me the specific resources mentioned in warnings

Ready to export? Run /export when ready.
```

**If PASS (no issues or INFO only)**:
```
✅ Excellent! Your data passed all validation checks.

Would you like to:
1. Export this data now (run /export)
2. Review the info suggestions for optional enhancements
3. Generate more data with /generate

What's next?
```

## Settings Integration

Check for validation settings in `.claude/cynthia-generator.local.md`:

```markdown
## Validation Settings
- Auto-validate after generate: yes|no (default: yes)
- Default severity level: all|warnings|critical (default: all)
- Strict mode: yes|no (default: no)
```

**Apply settings**:
- If user didn't specify severity, use "Default severity level" from settings
- If "Strict mode: yes", treat WARNINGS as CRITICAL (fail validation if any warnings)

**Strict Mode Behavior**:
```
⚠️  STRICT MODE ENABLED

Treating 2 warnings as critical errors.
Validation FAILED due to warnings (strict mode active).

To disable strict mode: Set "Strict mode: no" in .claude/cynthia-generator.local.md
```

## Error Handling

Handle these scenarios gracefully:

### File Not Found
```
✗ File not found: ./nonexistent.json

Please check the file path. Current directory: [pwd]

Available .json files in current directory:
- synthetic-patient-001.json
- test-output/patient-data.json

Try: /validate --file ./synthetic-patient-001.json
```

### Invalid JSON Syntax
```
✗ Invalid JSON format

Error: Unexpected token '}' at line 145
Unable to parse file - please fix JSON syntax first.

Tip: Use a JSON validator to identify syntax errors.
```

### Not a FHIR Bundle
```
✗ Not a FHIR Bundle

Found resourceType: "Patient"
Expected: "Bundle"

This command validates Bundles containing multiple resources.

To validate a single resource:
1. Wrap it in a Bundle:
   {
     "resourceType": "Bundle",
     "type": "collection",
     "entry": [{"resource": {your Patient resource}}]
   }
2. Then validate the Bundle

Or use this single resource in a /generate workflow.
```

### Empty Bundle
```
⚠️  Empty Bundle

Bundle contains no resources (entry array is empty).
Nothing to validate.

Try generating data first: /generate
```

### Agent Invocation Fails
```
✗ Validation agent unavailable

Unable to invoke fhir-data-validator agent.
Please try again or check plugin installation.

Tip: Ensure cynthia-generator plugin v0.2.0+ is installed.
```

### Partial Validation
```
⚠️  Validation partially completed

Successfully validated: 45/70 resources
Error encountered at: Observation/obs-complex-001

Partial results:
- Critical issues: 2
- Warnings: 3

Recommendation: Fix the error and re-run validation for complete results.
```

## Examples

### Example 1: Basic Validation
```
/validate --file ./synthetic-patient-001.json

✅ VALIDATION PASSED
- 70 resources validated
- 0 critical issues
- 0 warnings
- 2 info suggestions
```

### Example 2: Critical-Only Filter
```
/validate --file ./test-data.json --severity critical

✅ No critical issues found

(5 warnings and 3 info suggestions hidden)
Run with --severity all to see complete report.
```

### Example 3: Interactive Mode
```
/validate

> Which FHIR Bundle should I validate?
./patient-data.json

> Which severity levels should I report?
all

Running validation...

✓ VALIDATION PASSED
[detailed report...]
```

### Example 4: Validation Failure
```
/validate --file ./broken-data.json

❌ VALIDATION FAILED

Critical issues found:
1. Broken reference: Observation → Patient/missing-patient
2. Missing required field: Condition.code
3. Impossible value: Age -5 years

Recommendation: Fix these issues before using data.

Would you like help fixing these issues?
```

## Tips for Users

**Best Practices**:
- Run `/validate` before exporting to catch issues early
- Use `--severity critical` for quick sanity checks
- Use `--severity all` for comprehensive quality review
- Re-validate after manual JSON edits
- Check validation after adding custom conditions

**Performance**:
- Large bundles (500+ resources) may take 15-30 seconds
- Validation is read-only (never modifies your data)
- Can validate same file multiple times

**Custom Conditions**:
- Validation works with user-added diagnoses
- If clinical pattern defined: Full validation against your ranges
- If only ICD-10 mapping: Structural validation only
- Add patterns to `clinical-patterns.md` for complete validation

**Interpreting Results**:
- **CRITICAL**: Must fix (invalid structure, broken references)
- **WARNING**: Should review (unusual but possible values)
- **INFO**: Optional enhancements (suggestions for added realism)

## Integration with /generate

When auto-validation is enabled (default), `/generate` automatically runs validation:
- Brief summary shown after generation
- If critical issues: User alerted to run `/validate` for details
- If pass: User can proceed to export confidently

**Example auto-validation output**:
```
[after /generate completes]

✓ Validation complete: No critical issues found
  (2 minor suggestions - run /validate for details)

Would you like to export this data now?
```

## Future Enhancements (Out of Scope for v0.2.0)

Potential future features:
- Batch validation: `/validate --dir ./data/` (validate all .json files)
- Validation profiles: `/validate --profile research` (different rules for different use cases)
- HTML reports: `/validate --output report.html`
- Auto-fix suggestions: Agent offers to fix simple issues
- Continuous validation: Watch file for changes and re-validate

---

Remember: You are writing instructions FOR Claude, not TO the user. Use imperative form describing what Claude should do when executing this command.
