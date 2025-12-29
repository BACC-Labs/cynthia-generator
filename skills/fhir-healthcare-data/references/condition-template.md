# Custom Condition Template

Guide for adding new medical conditions to Cynthia Generator.

## Quick Start

To add a custom condition:
1. Add ICD-10 mapping to `icd10-mappings.md` (30 seconds)
2. Add clinical pattern to `clinical-patterns.md` using template below (5-15 minutes)
3. Test: `/generate --diagnosis "Your Condition"`
4. Validate: `/validate --file [output]`

## Step 1: Add ICD-10 Mapping

Edit `icd10-mappings.md` and add your condition to the appropriate category:

```markdown
## [Category Name]

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Your Condition Name | X00.0 | Brief description |
```

**Example**:
```markdown
## Neurological Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Essential Tremor | G25.0 | Involuntary trembling |
```

**Find ICD-10 codes**: https://www.icd10data.com/

## Step 2: Add Clinical Pattern

Edit `clinical-patterns.md` and add a new section using this template:

---

## [Condition Name] ([ICD-10 Code])

### Initial Presentation
**Timeline**: Day 0 (diagnosis visit)

**Encounter Type**: [Office Visit | Emergency Department | Hospital Admission]

**Presenting Symptoms/Observations**:
- [Symptom 1]
- [Symptom 2]
- [Symptom 3]

**Diagnostic Tests** (Observations):
- [Test Name]: [Normal Range] → [Abnormal Range indicating diagnosis]
- [LOINC Code if known]: [Value range at diagnosis]

**Vital Signs**:
- Blood Pressure: [Expected range] mmHg
- Heart Rate: [Expected range] bpm
- [Other relevant vitals]

**Initial Medications**:
- [First-line medication]: [Dosage] [Frequency]
- Alternative: [Second-line if first not tolerated]: [Dosage] [Frequency]

**Care Plan**:
- [Lifestyle modification 1]
- [Monitoring requirement 1]
- [Patient education topic]

### Follow-Up Pattern

**[Frequency] Intervals** (e.g., Monthly, 3-Month, 6-Month):
- **Encounter**: [Type of follow-up visit]
- **Observations**:
  - [What to monitor regularly]
  - [Lab tests and their frequency]
  - [Vital signs to track]
- **Medication Adjustments**:
  - If [condition X]: [Adjust medication how]
  - If [condition Y]: [Add second agent]

**Annual Screenings** (Procedures):
- [Annual screening 1]
- [Annual screening 2]

### Progression Scenarios

**Well-Controlled ([Define criteria, e.g., "Lab value <X"])**:
- **Timeline**: [Months X-Y]
- [Description of well-controlled state]
- [Medications maintained at current dose]
- [Visit frequency when stable]

**Moderately Controlled ([Define criteria])**:
- **Timeline**: [Months X-Y]
- [Description of moderate control]
- [Medication escalation approach]
- [Increased monitoring needs]

**Poorly Controlled ([Define criteria])**:
- **Timeline**: [Months X-Y]
- [Description of poor control]
- [Aggressive treatment approach]
- [Frequency of visits - typically monthly]
- [Risk of complications]

### Complications (Years X-Y+)

**[Complication Name] ([ICD-10 Code if applicable])**:
- [How this complication develops]
- [Diagnostic criteria for complication]
- [Treatment modifications needed]

---

## Complete Example: Essential Tremor (G25.0)

Here's a fully worked example to guide you:

---

## Essential Tremor (G25.0)

### Initial Presentation
**Timeline**: Day 0 (diagnosis visit)

**Encounter Type**: Office Visit (Neurology)

**Presenting Symptoms/Observations**:
- Bilateral hand tremor during voluntary movement
- Tremor worsens with stress, caffeine, or fatigue
- May affect head, voice, or legs in addition to hands
- Improves with alcohol consumption (classic diagnostic sign)
- No tremor at rest (distinguishes from Parkinson's disease)
- Family history positive in 50% of cases

**Diagnostic Tests** (Observations):
- Clinical diagnosis based on neurological examination
- Tremor frequency: 4-12 Hz (action tremor)
- Severity scale: 0-10 (initial presentation typically 4-7)
- Rule out hyperthyroidism: TSH 0.5-5.0 mIU/L (normal)
- Brain MRI: Normal findings (rules out structural causes)

**Vital Signs**:
- Blood Pressure: 110-130/70-85 mmHg (typically normal)
- Heart Rate: 70-90 bpm

**Initial Medications**:
- Propranolol 20mg PO BID (first-line), titrate to 80-320mg daily divided doses
- Alternative (if beta-blocker contraindicated): Primidone 50mg PO daily at bedtime, titrate to 250mg daily

**Care Plan**:
- Limit caffeine intake (coffee, tea, energy drinks)
- Stress reduction techniques (meditation, exercise)
- Occupational therapy evaluation for adaptive strategies
- Monitor medication efficacy and side effects

### Follow-Up Pattern

**Monthly Initially** (first 3 months while titrating medication):
- **Encounter**: Neurology follow-up
- **Observations**:
  - Tremor severity assessment (0-10 scale)
  - Functional impact evaluation (writing, eating, drinking, buttoning)
  - Blood pressure monitoring (propranolol can lower BP)
  - Heart rate (watch for bradycardia <60 bpm)
  - Weight (appetite changes possible with primidone)
- **Medication Adjustments**:
  - If tremor severity unchanged after 2 weeks: Increase propranolol by 20-40mg increments
  - If heart rate <60 bpm or systolic BP <100: Reduce propranolol or switch to primidone
  - If adequate tremor control achieved (severity 0-3): Maintain current dose

**Quarterly** (once medication stabilized):
- Tremor severity monitoring
- Medication adherence check
- Side effects assessment
- Functional status evaluation

**Annually**:
- Comprehensive neurological examination
- Reassess treatment effectiveness
- Liver function tests (if on primidone - can affect liver)
- Consider medication adjustment or alternative therapies if control inadequate

### Progression Scenarios

**Well-Controlled (Tremor severity 0-3/10)**:
- **Timeline**: Months 3-24
- Tremor minimally impacts daily activities
- Patient can perform fine motor tasks (writing, eating) with minimal difficulty
- Medication stable at 80-160mg propranolol daily or 125-250mg primidone daily
- Quarterly visits sufficient for monitoring
- Side effects minimal and tolerable

**Moderately Controlled (Tremor severity 4-6/10)**:
- **Timeline**: Months 6-36
- Tremor impacts some fine motor tasks but functional independence maintained
- Difficulty with handwriting, drinking from cup, threading needle
- Combination therapy considered: Propranolol 160mg + Primidone 125mg
- More frequent medication adjustments (every 1-2 months)
- Occupational therapy for adaptive devices helpful

**Poorly Controlled (Tremor severity 7-10/10)**:
- **Timeline**: Months 12+
- Tremor significantly impairs daily function
- Unable to feed self, write legibly, or perform fine motor tasks
- Maximum medical therapy inadequate (propranolol 320mg + primidone 250mg)
- Consider botulinum toxin injections for severe hand or head tremor
- Referral for deep brain stimulation (DBS) evaluation if candidate
- Monthly visits for close management
- Significant impact on quality of life

### Complications (Years 5-10+)

**Progressive Essential Tremor**:
- Tremor spreads from hands to head, voice, trunk over years
- Amplitude increases gradually (fine tremor becomes coarse)
- Development of intention tremor in addition to action tremor
- Social withdrawal and depression due to embarrassment
- Consider surgical interventions: Deep brain stimulation (thalamic DBS) or MRI-guided focused ultrasound thalamotomy

**Medication-Related Complications**:
- **Propranolol**: Fatigue, dizziness, bradycardia, hypotension, bronchospasm (if asthma), sexual dysfunction
- **Primidone**: Sedation, ataxia, nausea, confusion (especially in elderly), liver enzyme elevation
- Monitor closely and adjust dosage or switch medications as needed

---

## Validation After Adding Condition

After adding your condition to both mapping and clinical pattern files:

### Test Generation
```
/generate --patients 1 --diagnosis "Essential Tremor" --age-range 55-75 --timespan 2y
```

### Validate Output
```
/validate --file ./generated-tremor-patient.json
```

### Check Validation Report

**If pattern fully defined**:
```
✅ VALIDATION PASSED
- Clinical realism checks: Using Essential Tremor pattern ✓
- Tremor severity values within 0-10 range ✓
- Propranolol dosage appropriate (80mg BID) ✓
- Encounter frequency matches monthly → quarterly pattern ✓
```

**If only mapping added (no pattern)**:
```
⚠️  Custom condition detected: Essential Tremor (G25.0)
    No clinical pattern defined - structural validation only
    Recommendation: Add pattern to clinical-patterns.md for full validation

✓ VALIDATION PASSED (limited checks)
- FHIR structure valid ✓
- References valid ✓
- Dates chronological ✓
```

## Common Patterns by Specialty

### Cardiology Conditions
**Focus on**: BP, heart rate, EKG findings, cardiac enzymes (troponin, BNP), lipid panel

**Medications**: Beta-blockers, ACE inhibitors, ARBs, statins, antiplatelet agents (aspirin, clopidogrel)

**Procedures**: Cardiac catheterization, echocardiogram, stress test, EKG, Holter monitor

**Example Observations**:
- Troponin: <0.04 ng/mL (normal), >0.04 (cardiac injury)
- BNP: <100 pg/mL (normal), >100 (heart failure)

### Endocrine Conditions
**Focus on**: Hormone levels, glucose, thyroid function, electrolytes, calcium

**Medications**: Hormone replacement (levothyroxine, insulin), antidiabetics, bisphosphonates

**Procedures**: Lab monitoring (frequent), thyroid ultrasound, DEXA scan, endoscopy

**Example Observations**:
- TSH: 0.5-5.0 mIU/L (normal), <0.5 (hyperthyroid), >5.0 (hypothyroid)
- Free T4: 0.8-1.8 ng/dL

### Respiratory Conditions
**Focus on**: Oxygen saturation, respiratory rate, spirometry (FEV1, FVC), arterial blood gases

**Medications**: Bronchodilators (albuterol, tiotropium), corticosteroids (inhaled, oral), oxygen therapy

**Procedures**: Pulmonary function tests, chest X-ray, chest CT, bronchoscopy

**Example Observations**:
- O2 saturation: 95-100% (normal), <90% (hypoxemia)
- FEV1/FVC: >0.70 (normal), <0.70 (obstructive disease like COPD)

### Renal Conditions
**Focus on**: Creatinine, eGFR, BUN, electrolytes (K+, Na+, Ca2+, PO4), urinalysis

**Medications**: ACE inhibitors/ARBs (renal protection), phosphate binders, erythropoietin, dialysis

**Procedures**: Renal ultrasound, kidney biopsy, dialysis access creation

**Example Observations**:
- eGFR: >60 mL/min/1.73m² (normal), 30-59 (Stage 3 CKD), <15 (Stage 5, dialysis)
- Creatinine: 0.7-1.3 mg/dL (normal), >1.5 (elevated)

### Musculoskeletal Conditions
**Focus on**: Joint examination, range of motion, inflammatory markers (ESR, CRP), X-rays

**Medications**: NSAIDs, disease-modifying antirheumatic drugs (DMARDs like methotrexate), biologics, corticosteroids

**Procedures**: Joint aspiration, joint injection, X-rays, MRI, DEXA scan

**Example Observations**:
- ESR: <20 mm/hr (normal), >20 (inflammation)
- CRP: <3.0 mg/L (normal), >3.0 (inflammation)

## Dosing Reference

### Common Medication Dosing Patterns

**Antihypertensives**:
- ACE Inhibitors (lisinopril): 10-40mg daily
- ARBs (losartan): 25-100mg daily
- Beta-blockers (metoprolol): 25-200mg BID
- Calcium channel blockers (amlodipine): 2.5-10mg daily

**Antidiabetics**:
- Metformin: 500-2000mg daily (divided BID)
- Insulin (basal): 10-100 units daily (individualized)
- GLP-1 agonists (semaglutide): 0.25-1.0mg weekly

**Lipid Management**:
- Statins (atorvastatin): 10-80mg daily at bedtime
- Statins (simvastatin): 10-40mg daily

**Respiratory**:
- Albuterol inhaler: 2 puffs every 4-6 hours PRN
- Tiotropium (Spiriva): 18mcg daily via inhaler
- Prednisone (COPD/asthma exacerbation): 40-60mg daily × 5 days

## Value Ranges Reference

### Vital Signs (Normal Adult)
- Blood Pressure: 90-120/60-80 mmHg
- Heart Rate: 60-100 bpm
- Respiratory Rate: 12-20 breaths/min
- Temperature: 97.8-99.1°F (36.5-37.3°C)
- Oxygen Saturation: 95-100%
- BMI: 18.5-24.9 (normal), 25-29.9 (overweight), 30+ (obese)

### Common Labs
- Fasting Glucose: 70-100 mg/dL
- A1C: <5.7% (normal), 5.7-6.4% (prediabetes), ≥6.5% (diabetes)
- Total Cholesterol: <200 mg/dL (desirable)
- LDL: <100 mg/dL (optimal), <70 mg/dL (if high cardiac risk)
- HDL: >40 mg/dL (men), >50 mg/dL (women)
- Triglycerides: <150 mg/dL
- Creatinine: 0.7-1.3 mg/dL
- eGFR: >60 mL/min/1.73m²
- TSH: 0.5-5.0 mIU/L
- Hemoglobin: 13.5-17.5 g/dL (men), 12.0-15.5 g/dL (women)

## Tips for Creating Realistic Patterns

### Start Simple
Don't try to be perfect - approximate ranges are better than none:
- Define initial presentation clearly
- Add 2-3 follow-up scenarios
- Include 1-2 common medications
- Specify 2-3 key lab values to monitor

### Use Similar Conditions as Templates
Look at existing patterns in `clinical-patterns.md`:
- Chronic conditions (diabetes, hypertension) → good for other chronic diseases
- Acute conditions → good for infections, injuries
- Progressive conditions → good for degenerative diseases

### Be Consistent with Time Ranges
- Initial diagnosis = Day 0
- First follow-up = Weeks to months (depends on severity)
- Well-controlled = Longer intervals (quarterly, semi-annual, annual)
- Poorly controlled = Shorter intervals (weekly, monthly)

### Include Realistic Variations
Real patients don't all follow the same path:
- Some respond well to treatment (well-controlled scenario)
- Some struggle with adherence or side effects (moderate scenario)
- Some progress despite treatment (poorly controlled scenario)

## Resources

**ICD-10 Codes**:
- https://www.icd10data.com/ (search by diagnosis name)
- https://www.cms.gov/medicare/coding-billing/icd-10-codes

**Lab Codes (LOINC)**:
- https://loinc.org/ (search by test name)
- https://search.loinc.org/

**Medication Info**:
- https://reference.medscape.com/drugs (dosing reference)
- https://www.rxlist.com/ (drug information)

**Clinical Guidelines**:
- American Diabetes Association (diabetes): https://diabetesjournals.org/
- American College of Cardiology (cardiology): https://www.acc.org/
- GOLD Guidelines (COPD): https://goldcopd.org/
- Specialty society websites for condition-specific guidelines

## Questions?

**Not sure about ranges or patterns?**
1. Consult medical references or clinical practice guidelines for your specialty
2. Look at similar conditions already in `clinical-patterns.md`
3. Start with basic pattern; add detail iteratively
4. Test with `/generate` and `/validate` to refine

**Can't find ICD-10 code?**
- Search https://www.icd10data.com/ by diagnosis name
- Use most specific code available
- If multiple codes, choose primary code

**Need help with LOINC codes?**
- LOINC codes are optional
- Focus on observation name and value ranges
- Generator will work without LOINC codes

**Validation not working?**
- Ensure both ICD-10 mapping AND clinical pattern are added
- Check formatting matches existing patterns exactly
- Test generation first, then validate
- If validation still limited, pattern may not be recognized - check indentation and markdown formatting

---

## Summary Checklist

Before using your custom condition:

- [ ] Added to `icd10-mappings.md` with correct ICD-10 code
- [ ] Added to `clinical-patterns.md` with complete pattern
- [ ] Defined initial presentation with symptoms and diagnostic criteria
- [ ] Specified initial medications with dosages
- [ ] Defined follow-up pattern with encounter frequency
- [ ] Created progression scenarios (well/moderate/poor control)
- [ ] Tested with `/generate --diagnosis "Your Condition"`
- [ ] Validated output with `/validate`
- [ ] Reviewed validation report for accuracy

**You're ready!** Your custom condition is now available for synthetic data generation with full validation support.
