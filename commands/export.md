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
- `--destination json|csv|mysql|s3|fhir|api`: Export destination
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
   - **API (bacclabs.io)**: POST an existing local FHIR JSON file to the Cynthia API

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
- **Input file**: Path to FHIR bundle JSON file to convert (required)
- **Output directory**: Where to save CSV files (default: `./synthetic-data-csv/`)
- **Structure**: `by-type` (one CSV per resource type) or `flat` (single denormalized patient view)

### Native CSV Converter

When the user requests CSV export, execute the following Python script via the Bash tool. This is the authoritative CSV converter — do not attempt to write CSVs manually with the Write tool.

The script handles:
- JSON arrays of Bundle objects (e.g., `[{Bundle}, {Bundle}, ...]`)
- Single FHIR Bundle with nested patient bundles in `entry`
- Component observations (e.g., blood pressure systolic/diastolic)
- SDOH Z-code detection and `is_sdoh` / `sdoh_captured_in_assessment` flags
- Clinical note text extraction from Encounter.note
- Proper CSV escaping via Python's `csv.DictWriter`

```python
import json, csv, os, sys

# ── configuration ──────────────────────────────────────────────────────────────
INPUT_FILE  = "__INPUT_FILE__"   # replace with actual path
OUTPUT_DIR  = "__OUTPUT_DIR__"   # replace with actual path
NOTE_PREVIEW_LEN = 300           # max chars of encounter note in CSV

SDOH_PREFIXES = (
    "Z55","Z56","Z57","Z58","Z59","Z60","Z61","Z62","Z63",
    "Z64","Z65","Z72","Z73","Z74","Z75","Z76","Z91"
)

# ── helpers ─────────────────────────────────────────────────────────────────────
def first(lst, default=""):
    return lst[0] if lst else default

def get_icd(code_obj):
    """Return the ICD-10-CM coding from a FHIR CodeableConcept, else first coding."""
    codings = code_obj.get("coding", [])
    return next((c for c in codings if "icd-10" in c.get("system","").lower()), first(codings, {}))

def get_loinc(code_obj):
    codings = code_obj.get("coding", [])
    return next((c for c in codings if "loinc" in c.get("system","").lower()), first(codings, {}))

def is_sdoh_code(icd_code):
    return any(icd_code.startswith(p) for p in SDOH_PREFIXES)

def is_implied(condition_resource):
    """Detect implied-only SDOH: flagged in note text but not in assessment."""
    note_text = first(condition_resource.get("note",[{}]), {}).get("text","")
    return "IMPLIED" in note_text.upper() or "implied" in note_text.lower()

def note_text(resource):
    notes = resource.get("note", [])
    return first(notes, {}).get("text", "") if notes else ""

def write_csv(path, rows, fields):
    with open(path, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fields, extrasaction="ignore")
        writer.writeheader()
        writer.writerows(rows)
    return len(rows)

# ── load input ──────────────────────────────────────────────────────────────────
with open(INPUT_FILE, encoding="utf-8") as f:
    raw = json.load(f)

# normalise to a flat list of Bundle objects
if isinstance(raw, list):
    bundles = raw                          # JSON array of bundles
elif raw.get("resourceType") == "Bundle":
    # single outer bundle — each entry.resource may itself be a Bundle
    inner = [e.get("resource", e) for e in raw.get("entry", [])]
    bundles = [b for b in inner if b.get("resourceType") == "Bundle"] or [raw]
else:
    print("ERROR: input is not a FHIR Bundle or array of Bundles", file=sys.stderr)
    sys.exit(1)

os.makedirs(OUTPUT_DIR, exist_ok=True)

# ── accumulators ────────────────────────────────────────────────────────────────
patients, conditions, encounters, observations, medications, procedures = [], [], [], [], [], []

for bundle in bundles:
    bundle_id = bundle.get("id", "")
    entries   = bundle.get("entry", [])

    # index patient
    pat = next((e["resource"] for e in entries
                if e.get("resource",{}).get("resourceType") == "Patient"), {})
    pat_id    = pat.get("id", "")
    name_obj  = first(pat.get("name", [{}]), {})
    full_name = " ".join(name_obj.get("given", [])) + " " + name_obj.get("family", "")
    full_name = full_name.strip()
    addr      = first(pat.get("address", [{}]), {})

    patients.append({
        "patient_id"        : pat_id,
        "bundle_id"         : bundle_id,
        "mrn"               : next((i["value"] for i in pat.get("identifier",[]) if i.get("value")), ""),
        "full_name"         : full_name,
        "given_name"        : " ".join(name_obj.get("given", [])),
        "family_name"       : name_obj.get("family", ""),
        "gender"            : pat.get("gender", ""),
        "birth_date"        : pat.get("birthDate", ""),
        "address_use"       : addr.get("use", ""),
        "address_line"      : "; ".join(addr.get("line", [])),
        "city"              : addr.get("city", ""),
        "state"             : addr.get("state", ""),
        "postal_code"       : addr.get("postalCode", ""),
        "phone"             : next((t["value"] for t in pat.get("telecom",[])
                                    if t.get("system") == "phone"), ""),
        "email"             : next((t["value"] for t in pat.get("telecom",[])
                                    if t.get("system") == "email"), ""),
        "preferred_language": first(pat.get("communication",[{}]),{})
                                .get("language",{}).get("coding",[{}])[0].get("code","en-US")
                                if pat.get("communication") else "en-US",
        "marital_status"    : first(pat.get("maritalStatus",{})
                                .get("coding",[{}]),{}).get("display",""),
    })

    for entry in entries:
        r     = entry.get("resource", {})
        rtype = r.get("resourceType","")

        # ── Condition ──────────────────────────────────────────────────────────
        if rtype == "Condition":
            code_obj   = r.get("code", {})
            icd        = get_icd(code_obj)
            icd_code   = icd.get("code","")
            cat_code   = first(r.get("category",[{}]),{}).get("coding",[{}])[0].get("code","") \
                         if r.get("category") else ""
            sdoh       = is_sdoh_code(icd_code) or "health-concern" in cat_code
            implied    = is_implied(r) if sdoh else False
            conditions.append({
                "condition_id"               : r.get("id",""),
                "patient_id"                 : pat_id,
                "patient_name"               : full_name,
                "bundle_id"                  : bundle_id,
                "icd10_code"                 : icd_code,
                "icd10_display"              : icd.get("display", code_obj.get("text","")),
                "condition_text"             : code_obj.get("text",""),
                "category"                   : cat_code,
                "is_sdoh"                    : "Yes" if sdoh else "No",
                "sdoh_captured_in_assessment": "No" if implied else ("Yes" if sdoh else "N/A"),
                "clinical_status"            : first(r.get("clinicalStatus",{})
                                                 .get("coding",[{}]),{}).get("code",""),
                "verification_status"        : first(r.get("verificationStatus",{})
                                                 .get("coding",[{}]),{}).get("code",""),
                "onset_date"                 : r.get("onsetDateTime",""),
                "recorded_date"              : r.get("recordedDate",""),
                "note"                       : note_text(r)[:500],
            })

        # ── Encounter ──────────────────────────────────────────────────────────
        elif rtype == "Encounter":
            enc_type = first(r.get("type",[{}]),{})
            period   = r.get("period",{})
            nt       = note_text(r)
            encounters.append({
                "encounter_id"   : r.get("id",""),
                "patient_id"     : pat_id,
                "patient_name"   : full_name,
                "bundle_id"      : bundle_id,
                "status"         : r.get("status",""),
                "class_code"     : r.get("class",{}).get("code",""),
                "type_text"      : enc_type.get("text",""),
                "type_snomed"    : first(enc_type.get("coding",[{}]),{}).get("code",""),
                "period_start"   : period.get("start",""),
                "period_end"     : period.get("end",""),
                "service_provider": r.get("serviceProvider",{}).get("display",""),
                "note_preview"   : nt[:NOTE_PREVIEW_LEN],
                "note_full"      : nt,
            })

        # ── Observation ────────────────────────────────────────────────────────
        elif rtype == "Observation":
            code_obj   = r.get("code",{})
            loinc      = get_loinc(code_obj)
            cat_code   = first(r.get("category",[{}]),{}).get("coding",[{}])[0].get("code","") \
                         if r.get("category") else ""
            components = r.get("component",[])
            base = {
                "patient_id"   : pat_id,
                "patient_name" : full_name,
                "bundle_id"    : bundle_id,
                "status"       : r.get("status",""),
                "category"     : cat_code,
                "effective_date": r.get("effectiveDateTime",""),
                "encounter_ref": r.get("encounter",{}).get("reference",""),
            }
            if components:
                for comp in components:
                    cl  = get_loinc(comp.get("code",{}))
                    cvq = comp.get("valueQuantity",{})
                    observations.append({**base,
                        "observation_id" : r.get("id","") + "_" + cl.get("code",""),
                        "loinc_code"     : cl.get("code",""),
                        "loinc_display"  : cl.get("display",""),
                        "value"          : cvq.get("value",""),
                        "unit"           : cvq.get("unit",""),
                        "ucum_code"      : cvq.get("code",""),
                    })
            else:
                vq = r.get("valueQuantity",{})
                observations.append({**base,
                    "observation_id" : r.get("id",""),
                    "loinc_code"     : loinc.get("code",""),
                    "loinc_display"  : loinc.get("display", code_obj.get("text","")),
                    "value"          : vq.get("value",""),
                    "unit"           : vq.get("unit",""),
                    "ucum_code"      : vq.get("code",""),
                })

        # ── MedicationStatement ────────────────────────────────────────────────
        elif rtype == "MedicationStatement":
            med_obj = r.get("medicationCodeableConcept", r.get("medication",{}).get("CodeableConcept",{}))
            rx_code = first(med_obj.get("coding",[{}]),{}) if med_obj else {}
            dosage  = first(r.get("dosage",[{}]),{})
            medications.append({
                "medication_id"  : r.get("id",""),
                "patient_id"     : pat_id,
                "patient_name"   : full_name,
                "bundle_id"      : bundle_id,
                "rxnorm_code"    : rx_code.get("code",""),
                "medication_name": rx_code.get("display", med_obj.get("text","")),
                "status"         : r.get("status",""),
                "effective_start": r.get("effectivePeriod",{}).get("start",
                                   r.get("effectiveDateTime","")),
                "effective_end"  : r.get("effectivePeriod",{}).get("end",""),
                "dosage_text"    : dosage.get("text",""),
                "route"          : first(dosage.get("route",{}).get("coding",[{}]),{}).get("display",""),
                "note"           : note_text(r)[:200],
            })

        # ── Procedure ──────────────────────────────────────────────────────────
        elif rtype == "Procedure":
            code_obj = r.get("code",{})
            p_code   = first(code_obj.get("coding",[{}]),{})
            procedures.append({
                "procedure_id"    : r.get("id",""),
                "patient_id"      : pat_id,
                "patient_name"    : full_name,
                "bundle_id"       : bundle_id,
                "code"            : p_code.get("code",""),
                "code_system"     : p_code.get("system",""),
                "display"         : p_code.get("display", code_obj.get("text","")),
                "status"          : r.get("status",""),
                "performed_date"  : r.get("performedDateTime",
                                    r.get("performedPeriod",{}).get("start","")),
                "note"            : note_text(r)[:200],
            })

# ── write CSVs ──────────────────────────────────────────────────────────────────
results = {}

results["patients"] = write_csv(
    os.path.join(OUTPUT_DIR, "patients.csv"), patients,
    ["patient_id","bundle_id","mrn","full_name","given_name","family_name",
     "gender","birth_date","address_use","address_line","city","state",
     "postal_code","phone","email","preferred_language","marital_status"]
)
results["conditions"] = write_csv(
    os.path.join(OUTPUT_DIR, "conditions.csv"), conditions,
    ["condition_id","patient_id","patient_name","bundle_id","icd10_code",
     "icd10_display","condition_text","category","is_sdoh",
     "sdoh_captured_in_assessment","clinical_status","verification_status",
     "onset_date","recorded_date","note"]
)
results["encounters"] = write_csv(
    os.path.join(OUTPUT_DIR, "encounters.csv"), encounters,
    ["encounter_id","patient_id","patient_name","bundle_id","status",
     "class_code","type_text","type_snomed","period_start","period_end",
     "service_provider","note_preview","note_full"]
)
results["observations"] = write_csv(
    os.path.join(OUTPUT_DIR, "observations.csv"), observations,
    ["observation_id","patient_id","patient_name","bundle_id","loinc_code",
     "loinc_display","status","category","effective_date","encounter_ref",
     "value","unit","ucum_code"]
)
if medications:
    results["medications"] = write_csv(
        os.path.join(OUTPUT_DIR, "medications.csv"), medications,
        ["medication_id","patient_id","patient_name","bundle_id","rxnorm_code",
         "medication_name","status","effective_start","effective_end",
         "dosage_text","route","note"]
    )
if procedures:
    results["procedures"] = write_csv(
        os.path.join(OUTPUT_DIR, "procedures.csv"), procedures,
        ["procedure_id","patient_id","patient_name","bundle_id","code",
         "code_system","display","status","performed_date","note"]
    )

# ── SDOH summary ────────────────────────────────────────────────────────────────
sdoh_rows     = [c for c in conditions if c["is_sdoh"] == "Yes"]
captured      = [c for c in sdoh_rows if c["sdoh_captured_in_assessment"] == "Yes"]
implied_only  = [c for c in sdoh_rows if c["sdoh_captured_in_assessment"] == "No"]

print(f"Output directory : {OUTPUT_DIR}")
for name, count in results.items():
    print(f"  {name+'.csv':<20} {count} rows")
print(f"\nSDOH conditions  : {len(sdoh_rows)} of {len(conditions)} total")
print(f"  Captured in assessment : {len(captured)}")
print(f"  Implied (plan only)    : {len(implied_only)}")
```

### How to invoke from the export command

When the user requests CSV export, substitute the placeholders and run:

```bash
python3 << 'PYEOF'
# ... paste script above with INPUT_FILE and OUTPUT_DIR substituted ...
PYEOF
```

If Python 3 is not available, report the error and suggest the user install it.

### Output directory default

If the user did not specify an output directory, use:
- `./synthetic-data-csv/` relative to the input file's directory

If the input file is at `/path/to/sdoh-patients-10-bundle.json`, default output is `/path/to/synthetic-data-csv/`.

### CSV columns reference

**patients.csv** — one row per patient
| Column | Source |
|--------|--------|
| `patient_id` | `Patient.id` |
| `bundle_id` | parent Bundle id |
| `mrn` | `Patient.identifier[MR].value` |
| `full_name` | `given + family` |
| `given_name` / `family_name` | `Patient.name[0]` |
| `gender` | `Patient.gender` |
| `birth_date` | `Patient.birthDate` |
| `address_use` / `address_line` / `city` / `state` / `postal_code` | `Patient.address[0]` |
| `phone` / `email` | `Patient.telecom` |
| `preferred_language` | `Patient.communication[0].language` |
| `marital_status` | `Patient.maritalStatus.coding[0].display` |

**conditions.csv** — one row per condition (clinical and SDOH)
| Column | Source / Notes |
|--------|----------------|
| `icd10_code` / `icd10_display` | `Condition.code.coding[icd-10-cm]` |
| `is_sdoh` | `Yes` if code starts with Z55–Z76 or Z91, or category = health-concern |
| `sdoh_captured_in_assessment` | `Yes` if SDOH and NOT implied; `No` if implied (plan-only); `N/A` if clinical |
| `category` | `encounter-diagnosis` or `health-concern` |

**encounters.csv** — one row per encounter; includes `note_preview` (300 chars) and `note_full` (complete clinical note text)

**observations.csv** — one row per value; component observations (BP) produce one row per component

**medications.csv** / **procedures.csv** — present only when those resource types exist in the bundle

### Example Output:
```
Output directory : ./synthetic-data-csv/
  patients.csv         10 rows
  conditions.csv       63 rows
  encounters.csv       11 rows
  observations.csv     13 rows

SDOH conditions  : 30 of 63 total
  Captured in assessment : 12
  Implied (plan only)    : 18
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

---

## API (bacclabs.io) Export

When `--destination api` is specified, POST an existing local FHIR JSON file to the Cynthia API.

### Parameters to Gather:
- **Input file**: Path to a local FHIR bundle JSON file (required — ask if not provided)

### Credential Resolution:
1. Read `CYNTHIA_API_KEY` from the environment.
2. If not found in environment, check `.claude/cynthia-generator.local.md` for a line like `- API Key: <value>`.
3. If still not found: print setup instructions (same as `--mode api` in generate) and abort.

### Instructions:

1. **Read the FHIR bundle** from the specified local file.

2. **For each `Condition` resource** in the bundle's `entry` array, extract `code.coding[0].code` as the ICD-10 value. Use the first Condition found.

3. **POST to the Cynthia API**:
   ```bash
   curl -s -o /tmp/cynthia_resp.json -w "%{http_code}" \
     -X POST "${CYNTHIA_API_URL:-https://app.bacclabs.io}/api/v1/client/admissions/" \
     -H "Authorization: Api-Key $CYNTHIA_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"icd_code": "<extracted_icd10_code>"}'
   ```

4. **Handle response by HTTP status code** (same rules as `--mode api` in generate):

   - **201 Created**:
     ```
     ✅ Record pushed — admission #A{id}
     🔗 {dashboard_url}
     ```

   - **400 with `admission_limit`**: Display upgrade message, do not retry:
     ```
     ⚠️  Admission limit reached.
     Upgrade your plan: 🔗 https://app.bacclabs.io/billing
     ```

   - **401 Unauthorized**:
     ```
     ✗ Invalid API key. Check CYNTHIA_API_KEY and try again.
     ```

   - **Timeout or network error**: Print warning and do not crash:
     ```
     ⚠️  Network error — could not reach Cynthia API.
     ```

### Example Output:
```
✅ Record pushed — admission #A42
🔗 https://app.bacclabs.io/dashboard/acme/admissions/42
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
