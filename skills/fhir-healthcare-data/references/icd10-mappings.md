# ICD-10 Code Mappings

Common diagnoses mapped to ICD-10 codes for synthetic healthcare data generation.

## Endocrine/Metabolic Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Type 2 Diabetes Mellitus | E11.9 | Without complications |
| Type 2 Diabetes with Hyperglycemia | E11.65 | With hyperglycemia |
| Type 2 Diabetes with Nephropathy | E11.21 | Kidney complications |
| Type 2 Diabetes with Neuropathy | E11.40 | Nerve damage |
| Type 2 Diabetes with Retinopathy | E11.319 | Eye complications |
| Type 1 Diabetes Mellitus | E10.9 | Without complications |
| Prediabetes | R73.03 | Elevated glucose, not yet diabetic |
| Obesity | E66.9 | Unspecified |
| Morbid Obesity | E66.01 | BMI ≥40 |
| Hyperlipidemia | E78.5 | Elevated cholesterol/triglycerides |
| Metabolic Syndrome | E88.81 | Cluster of conditions |

## Cardiovascular Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Essential Hypertension | I10 | Primary high blood pressure |
| Coronary Artery Disease | I25.10 | Without angina |
| Coronary Artery Disease with Angina | I25.119 | With chest pain |
| Acute Myocardial Infarction | I21.9 | Heart attack |
| Congestive Heart Failure | I50.9 | Unspecified |
| Atrial Fibrillation | I48.91 | Irregular heart rhythm |
| Peripheral Vascular Disease | I73.9 | Unspecified |

## Respiratory Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Chronic Obstructive Pulmonary Disease (COPD) | J44.9 | Unspecified |
| Asthma | J45.909 | Unspecified, uncomplicated |
| Chronic Bronchitis | J42 | |
| Pneumonia | J18.9 | Unspecified organism |
| Sleep Apnea | G47.33 | Obstructive sleep apnea |

## Renal/Kidney Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Chronic Kidney Disease, Stage 1 | N18.1 | GFR ≥90 |
| Chronic Kidney Disease, Stage 2 | N18.2 | GFR 60-89 |
| Chronic Kidney Disease, Stage 3 | N18.3 | GFR 30-59 |
| Chronic Kidney Disease, Stage 4 | N18.4 | GFR 15-29 |
| Chronic Kidney Disease, Stage 5 | N18.5 | GFR <15 or dialysis |
| Acute Kidney Injury | N17.9 | Unspecified |

## Musculoskeletal Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Osteoarthritis | M19.90 | Unspecified site |
| Rheumatoid Arthritis | M06.9 | Unspecified |
| Osteoporosis | M81.0 | Age-related |
| Low Back Pain | M54.5 | |
| Gout | M10.9 | Unspecified |

## Mental Health Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Major Depressive Disorder | F33.9 | Recurrent |
| Generalized Anxiety Disorder | F41.1 | |
| Post-Traumatic Stress Disorder (PTSD) | F43.10 | |
| Bipolar Disorder | F31.9 | Unspecified |

## Gastrointestinal Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Gastroesophageal Reflux Disease (GERD) | K21.9 | Without esophagitis |
| Irritable Bowel Syndrome | K58.9 | Unspecified |
| Crohn's Disease | K50.90 | Unspecified |
| Ulcerative Colitis | K51.90 | Unspecified |
| Cirrhosis of Liver | K74.60 | Unspecified |

## Neurological Disorders

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Migraine | G43.909 | Unspecified, not intractable |
| Epilepsy | G40.909 | Unspecified, not intractable |
| Multiple Sclerosis | G35 | |
| Parkinson's Disease | G20 | |
| Alzheimer's Disease | G30.9 | Unspecified |

## Cancer/Oncology

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| Breast Cancer | C50.919 | Unspecified site |
| Lung Cancer | C34.90 | Unspecified |
| Colon Cancer | C18.9 | Unspecified |
| Prostate Cancer | C61 | |
| Leukemia | C95.90 | Unspecified |

## Infectious Diseases

| Diagnosis | ICD-10 Code | Notes |
|-----------|-------------|-------|
| HIV Disease | B20 | |
| Hepatitis C | B18.2 | Chronic viral |
| Tuberculosis | A15.9 | Respiratory, unspecified |
| Influenza | J11.1 | Unspecified, with respiratory manifestations |
| COVID-19 | U07.1 | |

## Usage Guidelines

### Natural Language to ICD-10

When user provides natural language diagnosis:
1. Search this table for matching diagnosis
2. Use the specific ICD-10 code provided
3. Note any relevant complications or specifications
4. Default to "unspecified" variant if details not provided

### ICD-10 to Natural Language

When user provides ICD-10 code:
1. Look up code in this table
2. Use diagnosis name for clinical pattern lookup
3. Consider code-specific details (e.g., E11.21 indicates nephropathy complication)

### Comorbidity Patterns

Common combinations to consider:
- **Metabolic Syndrome**: E88.81 + E11.9 (Diabetes) + I10 (Hypertension) + E66.9 (Obesity)
- **Cardiovascular Cluster**: I25.10 (CAD) + I10 (Hypertension) + E78.5 (Hyperlipidemia)
- **Diabetes Complications**: E11.9 + E11.21 (Nephropathy) + N18.3 (CKD)
- **Smoking-Related**: J44.9 (COPD) + I25.10 (CAD)

### Code Selection

When multiple codes available:
1. Start with unspecified code (e.g., E11.9 for diabetes)
2. Add specific complication codes as disease progresses
3. Use more specific codes when patient history provides details
4. Avoid overly specific codes without supporting clinical evidence
