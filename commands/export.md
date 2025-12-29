---
name: export
description: Export generated synthetic patient data to JSON, CSV, MySQL, S3, or FHIR servers
allowed-tools:
  - Read
  - Write
  - AskUserQuestion
  - Bash
argument-hint: "[--destination json|csv|mysql|s3|fhir] [--output PATH] [--format bundle|ndjson]"
---

# Export Synthetic Patient Data

Export generated FHIR-compliant synthetic healthcare data to various destinations including local files, databases, cloud storage, and FHIR servers.

## Overview

This command takes generated patient data and exports it to the destination of your choice. Supports multiple formats and destinations to integrate with existing healthcare systems and workflows.

## How to Use This Command

### Interactive Mode (Recommended)

Simply run:
```
/export
```

Claude will ask:
1. Export destination (JSON file, CSV, MySQL, S3, FHIR server)
2. Format preferences (Bundle, NDJSON, CSV structure)
3. Output location or connection details
4. Whether to create database tables automatically (MySQL)

### Command-Line Arguments Mode

Provide parameters upfront:
```
/export --destination json --output ./patient-data.json --format bundle
```

**Available Arguments**:
- `--destination json|csv|mysql|s3|fhir`: Export destination
- `--output PATH`: Output file path (for json/csv)
- `--format bundle|ndjson|flat`: Format for JSON export
- `--host HOST`: Database/FHIR server host (for mysql/fhir)
- `--database NAME`: Database name (for mysql)
- `--bucket NAME`: S3 bucket name (for s3)

## Instructions for Claude

When executing this command:

### Step 1: Check for Generated Data

Verify that patient data has been generated:
- Check if data exists in memory from previous `/generate` command
- If no data available, inform user they must run `/generate` first
- Confirm number of patients and resources to be exported

### Step 2: Gather Export Parameters

If arguments not provided, use AskUserQuestion to gather:

1. **Export Destination**:
   Ask: "Where would you like to export the data?"
   Options:
   - **JSON file**: Local file in JSON format
   - **CSV files**: Separate CSV files per resource type
   - **MySQL database**: Direct MySQL database insertion
   - **S3 bucket**: AWS S3 cloud storage
   - **FHIR server**: POST to FHIR R4 server endpoint

2. **Format** (for JSON/file exports):
   - **Bundle**: Single FHIR Bundle with all resources (recommended for FHIR servers)
   - **NDJSON**: Newline-delimited JSON (one resource per line)
   - **Separate files**: One file per patient or per resource type

3. **Destination-specific parameters** (see sections below)

### Step 3: Load Settings (if available)

Check for credentials and defaults in `.claude/cynthia-generator.local.md`:
- MySQL connection details (host, port, database, username, password)
- AWS S3 credentials (access key, secret key, bucket, region)
- FHIR server endpoint and authentication
- Default export format

If settings exist, use as defaults. Otherwise, prompt user for required details.

**Important**: Never commit credentials to version control. Always use local settings.

### Step 4: Execute Export

Follow destination-specific instructions below:

---

## JSON File Export

### Parameters to Gather:
- **Output path**: Where to save the file (default: `./synthetic-patient-data.json`)
- **Format**: Bundle, NDJSON, or separate files per patient

### Instructions:

**Bundle Format** (Single file with all resources):
```json
{
  "resourceType": "Bundle",
  "type": "collection",
  "entry": [
    {
      "resource": { /* Patient resource */ }
    },
    {
      "resource": { /* Condition resource */ }
    },
    // ... all resources
  ]
}
```

**NDJSON Format** (One resource per line):
```
{"resourceType":"Patient","id":"patient-001",...}
{"resourceType":"Condition","id":"condition-001",...}
{"resourceType":"Encounter","id":"encounter-001",...}
```

**Separate Files** (One Bundle per patient):
- Create directory: `./synthetic-data/`
- Save each patient as: `patient-{mrn}.json`

### Implementation:
1. Use Write tool to create JSON file(s)
2. Format JSON with proper indentation (for Bundle)
3. Validate JSON structure before writing
4. Report file location and size to user

### Example Output:
```
✓ Exported 10 patients to ./synthetic-patient-data.json
  - Format: FHIR Bundle (collection)
  - Size: 2.4 MB
  - Total resources: 700
```

---

## CSV File Export

### Parameters to Gather:
- **Output directory**: Where to save CSV files (default: `./synthetic-data-csv/`)
- **Structure**: One CSV per resource type or flattened patient view

### Instructions:

**One CSV per Resource Type**:
- `patients.csv`: Patient demographics
- `conditions.csv`: Diagnoses
- `encounters.csv`: Visits
- `observations.csv`: Labs and vitals
- `medications.csv`: Medication statements
- `procedures.csv`: Procedures

**Flattened Patient View** (denormalized):
- Single CSV with patient + latest observations + medications
- Useful for analytics and reporting

### CSV Structure for Resources:

**patients.csv**:
```csv
id,mrn,ssn,given_name,family_name,gender,birth_date,city,state,zip
patient-001,MRN123456,123-45-6789,John,Smith,male,1965-08-15,Boston,MA,02101
```

**conditions.csv**:
```csv
patient_id,condition_id,icd10_code,display,onset_date,status
patient-001,condition-001,E11.9,Type 2 Diabetes,2023-06-10,active
```

**observations.csv**:
```csv
patient_id,encounter_id,observation_id,loinc_code,display,value,unit,date
patient-001,enc-001,obs-001,4548-4,Hemoglobin A1C,8.2,%,2023-06-10
```

### Implementation:
1. Create output directory using Bash (mkdir -p)
2. Parse FHIR resources and extract fields
3. Use Write tool to create CSV files with headers
4. Ensure proper CSV escaping (quotes, commas)
5. Report created files to user

### Example Output:
```
✓ Exported 10 patients to ./synthetic-data-csv/
  Files created:
  - patients.csv (10 rows)
  - conditions.csv (23 rows)
  - encounters.csv (120 rows)
  - observations.csv (480 rows)
  - medications.csv (40 rows)
  - procedures.csv (30 rows)
```

---

## MySQL Database Export

### Parameters to Gather:
- **Host**: Database server (default: localhost)
- **Port**: Database port (default: 3306)
- **Database**: Database name
- **Username**: Database user
- **Password**: Database password (check settings first)
- **Auto-create tables**: Yes/No

### Instructions:

1. **Check Settings**:
   - Look for MySQL credentials in `.claude/cynthia-generator.local.md`
   - If not found, prompt user for connection details
   - **Security**: Warn user that password will be visible in command output

2. **Test Connection**:
   ```bash
   mysql -h HOST -P PORT -u USERNAME -pPASSWORD -e "SELECT 1"
   ```
   - If fails, report error and request corrected credentials

3. **Ask About Table Creation**:
   - "Should I auto-create tables if they don't exist?"
   - If Yes: Create FHIR-based table schema
   - If No: Assume tables exist, insert into them

4. **Create Tables** (if auto-create chosen):

**patients table**:
```sql
CREATE TABLE IF NOT EXISTS patients (
  id VARCHAR(255) PRIMARY KEY,
  mrn VARCHAR(50) UNIQUE,
  ssn VARCHAR(11),
  given_name VARCHAR(100),
  family_name VARCHAR(100),
  gender VARCHAR(20),
  birth_date DATE,
  address_line VARCHAR(255),
  city VARCHAR(100),
  state VARCHAR(2),
  postal_code VARCHAR(10),
  phone VARCHAR(20),
  email VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**conditions table**:
```sql
CREATE TABLE IF NOT EXISTS conditions (
  id VARCHAR(255) PRIMARY KEY,
  patient_id VARCHAR(255),
  icd10_code VARCHAR(10),
  display VARCHAR(255),
  onset_date DATE,
  clinical_status VARCHAR(50),
  verification_status VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (patient_id) REFERENCES patients(id)
);
```

Create similar tables for:
- encounters
- observations
- medications
- procedures
- allergies
- immunizations
- diagnostic_reports
- care_plans

5. **Insert Data**:
   - Parse FHIR resources
   - Extract relevant fields
   - Build INSERT statements
   - Execute using Bash + mysql client
   - Use transactions for safety (BEGIN, COMMIT, ROLLBACK on error)

6. **Validate Insertion**:
   - Query row counts: `SELECT COUNT(*) FROM patients`
   - Report success or errors

### Implementation Example:
```bash
mysql -h localhost -u root -ppassword healthcare_test <<EOF
BEGIN;
INSERT INTO patients (id, mrn, given_name, family_name, gender, birth_date, city, state)
VALUES ('patient-001', 'MRN123456', 'John', 'Smith', 'male', '1965-08-15', 'Boston', 'MA');
COMMIT;
EOF
```

### Example Output:
```
✓ Connected to MySQL database: healthcare_test@localhost
✓ Created tables: patients, conditions, encounters, observations, medications, procedures
✓ Inserted data:
  - 10 patients
  - 23 conditions
  - 120 encounters
  - 480 observations
  - 40 medications
  - 30 procedures
✓ Export complete
```

---

## S3 Bucket Export

### Parameters to Gather:
- **Bucket name**: S3 bucket
- **Region**: AWS region (default: us-east-1)
- **Prefix/Path**: S3 key prefix (default: synthetic-data/)
- **AWS Credentials**: Access key and secret key (check settings)
- **Format**: JSON bundle or NDJSON

### Instructions:

1. **Check Settings**:
   - Look for AWS credentials in `.claude/cynthia-generator.local.md`
   - If not found, prompt user for:
     - AWS_ACCESS_KEY_ID
     - AWS_SECRET_ACCESS_KEY
     - Bucket name
     - Region

2. **Generate Data Files**:
   - Create temporary local files with generated data
   - Use JSON Bundle or NDJSON format

3. **Upload to S3**:
   ```bash
   AWS_ACCESS_KEY_ID=xxx AWS_SECRET_ACCESS_KEY=yyy aws s3 cp \
     ./temp-data.json \
     s3://bucket-name/synthetic-data/patients-20240115.json \
     --region us-east-1
   ```

4. **Verify Upload**:
   ```bash
   aws s3 ls s3://bucket-name/synthetic-data/
   ```

5. **Clean Up**:
   - Remove temporary local files

### Implementation:
- Use Bash tool to execute AWS CLI commands
- Ensure aws CLI is installed (check with `which aws`)
- Handle authentication via environment variables
- Report S3 URI to user

### Example Output:
```
✓ Uploaded to S3:
  - s3://healthcare-test-data/synthetic-data/patients-2024-01-15.json
  - Size: 2.4 MB
  - Region: us-east-1
✓ Export complete
```

---

## FHIR Server Export

### Parameters to Gather:
- **FHIR Server Endpoint**: Base URL (e.g., https://fhir.example.com/fhir)
- **Authentication**: Bearer token, Basic auth, or API key
- **Method**: Batch transaction or individual POST
- **Credentials**: Check settings

### Instructions:

1. **Check Settings**:
   - Look for FHIR server endpoint in `.claude/cynthia-generator.local.md`
   - Check for auth token

2. **Validate Endpoint**:
   ```bash
   curl -X GET https://fhir.example.com/fhir/metadata
   ```
   - Should return CapabilityStatement
   - Verify it's FHIR R4 server

3. **Choose Method**:
   - **Batch Transaction** (recommended): Single POST with Bundle
   - **Individual Resources**: POST each resource separately

4. **Batch Transaction Approach**:
   - Create transaction Bundle:
     ```json
     {
       "resourceType": "Bundle",
       "type": "transaction",
       "entry": [{
         "request": {
           "method": "POST",
           "url": "Patient"
         },
         "resource": { /* Patient resource */ }
       }, {
         "request": {
           "method": "POST",
           "url": "Condition"
         },
         "resource": { /* Condition resource */ }
       }]
     }
     ```
   - POST to server:
     ```bash
     curl -X POST https://fhir.example.com/fhir \
       -H "Content-Type: application/fhir+json" \
       -H "Authorization: Bearer TOKEN" \
       -d @transaction-bundle.json
     ```

5. **Handle Response**:
   - Check for 200/201 status codes
   - Parse response for created resource IDs
   - Report any errors

6. **Individual POST Approach** (fallback):
   - POST each resource separately
   - Handle dependencies (Patient before Condition)
   - Slower but more compatible

### Implementation:
- Use Bash + curl for HTTP requests
- Handle authentication headers
- Parse JSON responses
- Report success/errors per resource

### Example Output:
```
✓ Posted to FHIR server: https://fhir.example.com/fhir
  - Method: Transaction Bundle
  - Resources created: 700
  - Patients: 10
  - Conditions: 23
  - Encounters: 120
  - Observations: 480
  - Medications: 40
  - Procedures: 30
✓ Export complete
```

---

## Error Handling

Handle these scenarios gracefully:

### No Generated Data:
```
✗ No patient data available to export.
  Please run /generate first to create synthetic data.
```

### Invalid Destination:
```
✗ Invalid destination: xyz
  Valid options: json, csv, mysql, s3, fhir
```

### Connection Failures (MySQL, S3, FHIR):
```
✗ Failed to connect to MySQL database
  Error: Access denied for user 'root'@'localhost'
  Please check credentials in .claude/cynthia-generator.local.md
```

### Missing Credentials:
```
✗ AWS credentials not found
  Please configure in .claude/cynthia-generator.local.md or provide via arguments
```

### File Write Errors:
```
✗ Failed to write to ./data.json
  Error: Permission denied
  Please check directory permissions
```

## Security Considerations

### Credential Storage:
- **Always** store credentials in `.claude/cynthia-generator.local.md`
- **Never** hardcode credentials in commands or code
- **Never** commit credentials to git (gitignored)
- Warn user if providing credentials via command line (visible in history)

### Connection Security:
- **MySQL**: Use SSL connections when possible
- **S3**: Use AWS IAM roles when available
- **FHIR**: Require HTTPS endpoints
- Validate server certificates

## Settings File Format

Example `.claude/cynthia-generator.local.md`:

```markdown
# Cynthia Generator Settings

## MySQL Database
- Host: localhost
- Port: 3306
- Database: healthcare_test
- Username: dev_user
- Password: secure_password_here

## AWS S3
- Access Key: AKIAIOSFODNN7EXAMPLE
- Secret Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
- Bucket: synthetic-health-data
- Region: us-east-1

## FHIR Server
- Endpoint: https://fhir.example.com/fhir
- Auth Token: bearer_token_here

## Defaults
- Default Export Format: bundle
- Default Destination: json
```

## Examples

### Example 1: Export to JSON
```
/export
> Destination? json
> Output path? ./patient-data.json
> Format? bundle
✓ Exported to ./patient-data.json (2.4 MB)
```

### Example 2: Export to MySQL
```
/export --destination mysql --database healthcare_test
> Auto-create tables? yes
✓ Created tables and inserted 700 resources
```

### Example 3: Export to S3
```
/export --destination s3 --bucket my-test-data
✓ Uploaded to s3://my-test-data/synthetic-data/patients-2024-01-15.json
```

## Tips

- Use Bundle format for FHIR server compatibility
- Use CSV format for analytics and reporting tools
- Use NDJSON for streaming or large datasets
- Test database credentials before large exports
- Check S3 bucket permissions before upload
- Validate FHIR server compatibility (R4 vs STU3)

---

Remember: You are writing instructions FOR Claude, not TO the user. Use imperative form describing what Claude should do when executing this command.
