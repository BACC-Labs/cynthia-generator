# Cynthia Generator

Generate synthetic FHIR-compliant healthcare data for EHR testing without exposing real patient information.

## Overview

Cynthia Generator is a Claude Code plugin that creates realistic longitudinal patient health records following FHIR specifications. Perfect for healthcare developers who need test data but can't use real patient information due to HIPAA and privacy regulations.

## Features

- **FHIR-Compliant**: Generates data following FHIR R4 specifications
- **Diagnosis-Driven**: Creates realistic patient histories based on primary/secondary diagnoses
- **ICD-10 Support**: Accepts both natural language ("Type 2 Diabetes") and ICD-10 codes ("E11.9")
- **Longitudinal Data**: Generates multi-year patient histories with realistic clinical patterns
- **Data Validation** *(v0.2.0+)*: Automatic quality checks for clinical realism and referential integrity
- **Extensible** *(v0.2.0+)*: Add custom medical conditions by editing markdown templates
- **Multiple Export Options**: JSON, CSV, MySQL, S3, FHIR servers
- **Secure**: Credentials stored locally only, never committed to version control

## Installation

```bash
claude plugin install https://github.com/BACC-Labs/cynthia-generator
```

Or install from local directory:

```bash
claude plugin install /path/to/cynthia-generator
```

## Prerequisites

Depending on your export destination, you may need:

- **MySQL Export**: MySQL client installed and accessible
- **S3 Export**: AWS credentials configured
- **FHIR Server Export**: Endpoint URL and authentication credentials

## Quick Start

### Generate Synthetic Patient Data

```bash
/generate
```

Claude will interactively ask you:
- Number of patients to generate
- Primary diagnosis (natural language or ICD-10 code)
- Demographics (age range, gender, ethnicity)
- Time span for patient history
- Whether to export immediately

### Validate Generated Data *(v0.2.0+)*

```bash
/validate --file ./synthetic-patient-001.json
```

Check your generated data for:
- **Clinical realism** (appropriate lab values, medications, disease progression)
- **Referential integrity** (all resource references are valid)
- **FHIR compliance** (required fields, proper code systems)

Auto-validation runs after `/generate` by default.

### Export Data

```bash
/export
```

Choose from:
- **Local files** (JSON, CSV)
- **MySQL database** (auto-create tables or use existing schema)
- **S3 bucket**
- **FHIR server endpoint**

## Configuration (Optional)

Create `.claude/cynthia-generator.local.md` to store credentials and preferences:

```markdown
# Cynthia Generator Configuration

## MySQL Database
- Host: localhost
- Port: 3306
- Database: healthcare_test
- Username: dev_user
- Password: your_password

## AWS S3
- Access Key: YOUR_ACCESS_KEY
- Secret Key: YOUR_SECRET_KEY
- Bucket: synthetic-health-data
- Region: us-east-1

## FHIR Server
- Endpoint: https://your-fhir-server.com/fhir
- Auth Token: your_token_here

## Defaults
- Default Patients: 1
- Default Time Span: 1 year
- Default Export Format: JSON

## Validation Settings
- Auto-validate after generate: yes
- Default severity level: all
- Strict mode: no
```

**Important**: This file is gitignored and stays local to your machine.

## FHIR Resources Supported

- **Patient** (demographics, identifiers)
- **Encounter** (visits, admissions, appointments)
- **Observation** (vitals, lab results)
- **Condition** (diagnoses)
- **Medication** (prescriptions, medication statements)
- **Procedure** (surgeries, treatments)
- **AllergyIntolerance** (allergies)
- **Immunization** (vaccinations)
- **DiagnosticReport** (lab reports, imaging)
- **CarePlan** (treatment plans)

## Adding Custom Medical Conditions *(v0.2.0+)*

Extend the plugin with your own medical conditions by editing markdown templates.

### Quick Process

1. **Add ICD-10 Mapping**: Edit `skills/fhir-healthcare-data/references/icd10-mappings.md`
   ```markdown
   | Your Condition | X00.0 | Brief description |
   ```

2. **Add Clinical Pattern**: Edit `skills/fhir-healthcare-data/references/clinical-patterns.md`
   - Define initial presentation with symptoms and tests
   - Specify medications and dosages
   - Create follow-up patterns
   - Include progression scenarios

3. **Test Generation**:
   ```bash
   /generate --diagnosis "Your Condition"
   ```

4. **Validate Output**:
   ```bash
   /validate --file ./generated-patient.json
   ```

### Detailed Template Guide

See `skills/fhir-healthcare-data/references/condition-template.md` for:
- **Complete template structure** for clinical patterns
- **Worked example**: Essential Tremor (G25.0) with full implementation
- **Common patterns by specialty**: Cardiology, Endocrine, Respiratory, Renal, Musculoskeletal
- **Lab ranges and medication dosing** references
- **Validation tips** for custom conditions

### Pre-Defined Conditions

The plugin includes clinical patterns for:
- Type 2 Diabetes (E11.9)
- Hypertension (I10)
- COPD (J44.9)
- Coronary Artery Disease (I25.10)
- Chronic Kidney Disease Stage 3 (N18.3)
- Congestive Heart Failure (I50.9)
- Asthma (J45.909)

Plus 60+ ICD-10 mappings across 10 categories.

## Example Use Cases

**Generate diabetes patient data**:
```
/generate
> How many patients? 10
> Primary diagnosis? Type 2 Diabetes
> Age range? 45-65
> Time span? 3 years
```

**Export to MySQL**:
```
/export
> Destination? MySQL
> Auto-create tables? Yes
```

**Generate using ICD-10 code**:
```
/generate
> Primary diagnosis? E11.9
```

## Security & Privacy

- All credentials stored in `.claude/*.local.md` (gitignored)
- No data sent to external services (local generation)
- HIPAA-compliant synthetic data generation
- No real patient information used

## Contributing

Contributions welcome! Please open issues or pull requests on GitHub.

## License

MIT

## Links

- **Website**: https://bacclabs.io
- **GitHub**: https://github.com/BACC-Labs/cynthia-generator
- **Issues**: https://github.com/BACC-Labs/cynthia-generator/issues

---

Built with [Claude Code](https://claude.com/claude-code)
