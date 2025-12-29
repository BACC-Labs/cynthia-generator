# Cynthia Generator

Generate synthetic FHIR-compliant healthcare data for EHR testing without exposing real patient information.

## Overview

Cynthia Generator is a Claude Code plugin that creates realistic longitudinal patient health records following FHIR specifications. Perfect for healthcare developers who need test data but can't use real patient information due to HIPAA and privacy regulations.

## Features

- **FHIR-Compliant**: Generates data following FHIR R4 specifications
- **Diagnosis-Driven**: Creates realistic patient histories based on primary/secondary diagnoses
- **ICD-10 Support**: Accepts both natural language ("Type 2 Diabetes") and ICD-10 codes ("E11.9")
- **Longitudinal Data**: Generates multi-year patient histories with realistic clinical patterns
- **Multiple Export Options**: JSON, CSV, MySQL, S3, FHIR servers
- **Secure**: Credentials stored locally only, never committed to version control

## Installation

```bash
claude plugin install https://github.com/bacclabs/cynthia-generator
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
- **GitHub**: https://github.com/bacclabs/cynthia-generator
- **Issues**: https://github.com/bacclabs/cynthia-generator/issues

---

Built with [Claude Code](https://claude.com/claude-code)
