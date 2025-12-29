# Validation Test Data

This directory contains test FHIR Bundles for validating the v0.2.0 validation functionality.

## Test Files

### valid-patient.json
**Expected Result**: ✅ VALIDATION PASSED

Perfect data with no issues:
- All required fields present
- All references valid (Patient → Condition, Encounter, Observations, Medication)
- Clinical values within normal ranges:
  - A1C: 7.8% (typical for new diabetes diagnosis)
  - BP: 128/82 mmHg (normal range)
  - Metformin: 500mg BID (appropriate first-line therapy)
- Chronological dates
- Proper code systems (ICD-10, LOINC, RxNorm)

### critical-issues.json
**Expected Result**: ❌ VALIDATION FAILED

Data with 4 critical issues:

1. **Missing required field**: Condition missing `code` field (no ICD-10 code)
2. **Broken reference**: Observation references non-existent Patient (`Patient/patient-DOES-NOT-EXIST`)
3. **Impossible value**: Blood pressure -50/30 mmHg (negative systolic is impossible)
4. **Impossible value**: A1C 25.5% (medically impossible - incompatible with life)
5. **Impossible dosage**: Aspirin 50000mg daily (toxic - normal is 81-325mg)

### warnings.json
**Expected Result**: ✓ VALIDATION PASSED (with warnings)

Data with 3 warnings but no critical issues:

1. **WARNING - Unusually high A1C**: 13.2% (extremely high but medically possible)
   - Typical range at diagnosis: 6.5-9.0%
   - This indicates very poor glycemic control
2. **WARNING - Low blood pressure**: 95/58 mmHg (hypotensive but possible)
   - Normal range: 90-120/60-80 mmHg
   - This is at lower bound, concerning but not impossible
3. **WARNING - Very high glucose**: 385 mg/dL (severely elevated)
   - Normal fasting: 70-100 mg/dL
   - This correlates with the high A1C but is unusually high

All references are valid, all required fields present, all values are medically possible (though extreme).

## Usage

### Test Valid Data
```bash
/validate --file ./test-output/validation-tests/valid-patient.json
```

Expected: No issues found, full pass

### Test Critical Issues Detection
```bash
/validate --file ./test-output/validation-tests/critical-issues.json
```

Expected: 5 critical issues flagged, validation fails

### Test Warning Detection
```bash
/validate --file ./test-output/validation-tests/warnings.json
```

Expected: 3 warnings flagged, validation passes with warnings

### Test Severity Filtering
```bash
/validate --file ./test-output/validation-tests/warnings.json --severity critical
```

Expected: No critical issues shown (warnings hidden)

## Validation Categories Tested

### Clinical Realism
- ✅ Normal lab values (valid-patient.json)
- ❌ Impossible values (critical-issues.json: A1C 25.5%, BP -50/30)
- ⚠️ Extreme values (warnings.json: A1C 13.2%, glucose 385)
- ✅ Appropriate medications and dosages (valid-patient.json)
- ❌ Toxic dosages (critical-issues.json: aspirin 50000mg)

### Referential Integrity
- ✅ All references valid (valid-patient.json, warnings.json)
- ❌ Broken patient reference (critical-issues.json)

### Structural Validation
- ✅ All required fields (valid-patient.json, warnings.json)
- ❌ Missing required field (critical-issues.json: Condition.code)
- ✅ Proper code systems (all files use ICD-10, LOINC, RxNorm)
- ✅ Chronological dates (all files)

## Expected Validation Behavior

### Auto-Validation (after /generate)
When generating new data, auto-validation should:
- Show brief summary: "✓ No critical issues" or "⚠️ 3 critical issues found"
- Recommend running `/validate` for details if issues found

### Manual Validation (/validate command)
Shows detailed report with:
- Summary (resource count, issue breakdown by severity)
- Issue details (resource ID, description, recommendation)
- Severity filtering (--severity all|warnings|critical)
- Next step suggestions (fix issues, export, or review details)

## Extending Tests

To add more test scenarios:
1. Create new .json file in this directory
2. Add description to this README
3. Test with `/validate --file ./test-output/validation-tests/your-test.json`
4. Document expected results
