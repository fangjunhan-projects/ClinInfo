# openFDA API Schema

## Base URL
```
https://api.fda.gov
```

## Available Endpoints

| Endpoint | URL | Description |
|----------|-----|-------------|
| Drug Adverse Events | `/drug/event.json` | Reports from FDA Adverse Event Reporting System (FAERS) |
| Drugs@FDA | `/drug/drugsfda.json` | FDA-approved drug products since 1939 |
| Drug Labeling | `/drug/label.json` | Structured product labeling for approved drugs |
| Drug NDC Directory | `/drug/ndc.json` | National Drug Code directory |
| Drug Recall Enforcement | `/drug/enforcement.json` | Drug product recall data |

## Request Method
`GET`

---

## Query Parameters (Inputs)

### Search Parameter
- **Parameter**: `search`
- **Type**: `string`
- **Description**: Filter results by searching specific fields. Syntax: `search=field:term`
- **Operators**:
  - `+AND+` — combine terms with AND logic
  - `+` (space) — combine terms with OR logic
  - `""` — exact phrase match
  - `[value1+TO+value2]` — range (dates, numbers, strings)
- **Examples**:
  - `search=patient.drug.medicinalproduct:aspirin`
  - `search=patient.drug.openfda.brand_name:"ADVIL"+AND+serious:1`
  - `search=receivedate:[20200101+TO+20201231]`

### Limit Parameter
- **Parameter**: `limit`
- **Type**: `integer`
- **Description**: Number of records to return
- **Default**: `1`
- **Maximum**: `1000`
- **Example**: `limit=10`

### Skip Parameter
- **Parameter**: `skip`
- **Type**: `integer`
- **Description**: Number of records to skip (for pagination)
- **Default**: `0`
- **Maximum**: `25000`
- **Example**: `skip=10`

### Sort Parameter
- **Parameter**: `sort`
- **Type**: `string`
- **Description**: Sort results by a field in ascending or descending order
- **Modifiers**: `:asc` or `:desc`
- **Example**: `sort=receivedate:desc`

### Count Parameter
- **Parameter**: `count`
- **Type**: `string`
- **Description**: Instead of returning records, count unique values of a field. Use `.exact` suffix for full phrase counting.
- **Example**: `count=patient.reaction.reactionmeddrapt.exact`

### API Key (Optional)
- **Parameter**: `api_key`
- **Type**: `string`
- **Description**: Authentication key for higher rate limits. Not required but recommended for regular use.
- **Example**: `api_key=YOUR_API_KEY`

---

## Endpoint 1: Drug Adverse Events

### Base URL
```
https://api.fda.gov/drug/event.json
```

### Data Source
FDA Adverse Event Reporting System (FAERS). Covers records from 2004 onward, updated quarterly with ~3 month lag.

### Key Searchable Fields

#### Report Fields
- `safetyreportid` — Unique report identifier (e.g., `6176304-1`)
- `receivedate` — Date FDA received the report (format: `YYYYMMDD`)
- `receiptdate` — Date of most recent information for the report
- `serious` — `1` = serious, `2` = not serious
- `seriousnesscongenitalanomali` — `1` if congenital anomaly
- `seriousnessdeath` — `1` if death reported
- `seriousnessdisabling` — `1` if disabling
- `seriousnesshospitalization` — `1` if hospitalization
- `seriousnesslifethreatening` — `1` if life threatening
- `seriousnessother` — `1` if other serious outcome
- `reporttype` — `1` = Expedited, `2` = Periodic, `3` = Periodic (delayed), `4` = IND Safety Report
- `occurcountry` — Country where event occurred (ISO 2-letter code)
- `primarysourcecountry` — Country of primary reporter

#### Patient Fields
- `patient.patientsex` — `0` = Unknown, `1` = Male, `2` = Female
- `patient.patientonsetage` — Age at onset
- `patient.patientweight` — Weight in kg
- `patient.reaction.reactionmeddrapt` — Adverse reaction term (MedDRA preferred term)
- `patient.reaction.reactionoutcome` — `1` = Recovered, `2` = Recovering, `3` = Not recovered, `4` = Recovered with sequelae, `5` = Fatal, `6` = Unknown

#### Drug Fields
- `patient.drug.medicinalproduct` — Drug name as reported
- `patient.drug.drugcharacterization` — `1` = Suspect, `2` = Concomitant, `3` = Interacting
- `patient.drug.drugindication` — Indication for drug use
- `patient.drug.drugdosagetext` — Dosage information as text
- `patient.drug.drugadministrationroute` — Route code (e.g., `048` = Oral)
- `patient.drug.actiondrug` — Action taken: `1` = Withdrawn, `2` = Dose reduced, `3` = Dose increased, `4` = Dose not changed, `5` = Unknown, `6` = Not applicable

#### openFDA Harmonized Fields (on drugs)
- `patient.drug.openfda.brand_name` — Brand name(s)
- `patient.drug.openfda.generic_name` — Generic name(s)
- `patient.drug.openfda.manufacturer_name` — Manufacturer(s)
- `patient.drug.openfda.product_type` — e.g., `HUMAN PRESCRIPTION DRUG`
- `patient.drug.openfda.route` — Route(s) of administration
- `patient.drug.openfda.substance_name` — Active substance(s)
- `patient.drug.openfda.rxcui` — RxNorm concept ID(s)
- `patient.drug.openfda.pharm_class_epc` — Pharmacologic class (Established Pharmacologic Class)
- `patient.drug.openfda.application_number` — NDA/ANDA number(s)
- `patient.drug.openfda.unii` — Unique Ingredient Identifier(s)

### Response Schema

```json
{
  "meta": {
    "disclaimer": "string",
    "terms": "string",
    "license": "string",
    "last_updated": "string (YYYY-MM-DD)",
    "results": {
      "skip": "integer",
      "limit": "integer",
      "total": "integer"
    }
  },
  "results": [AdverseEventReport]
}
```

### Adverse Event Report Object

```json
{
  "safetyreportversion": "string",
  "safetyreportid": "string",
  "primarysourcecountry": "string (ISO 2-letter)",
  "occurcountry": "string (ISO 2-letter)",
  "transmissiondate": "string (YYYYMMDD)",
  "reporttype": "string (1|2|3|4)",
  "serious": "string (1|2)",
  "seriousnessdeath": "string (1, if applicable)",
  "seriousnesshospitalization": "string (1, if applicable)",
  "seriousnesslifethreatening": "string (1, if applicable)",
  "seriousnessdisabling": "string (1, if applicable)",
  "seriousnesscongenitalanomali": "string (1, if applicable)",
  "seriousnessother": "string (1, if applicable)",
  "receivedate": "string (YYYYMMDD)",
  "receiptdate": "string (YYYYMMDD)",
  "fulfillexpeditecriteria": "string",
  "companynumb": "string",
  "primarysource": {
    "reportercountry": "string",
    "qualification": "string (1=Physician, 2=Pharmacist, 3=Other health professional, 4=Lawyer, 5=Consumer)"
  },
  "sender": {
    "sendertype": "string",
    "senderorganization": "string"
  },
  "receiver": {
    "receivertype": "string",
    "receiverorganization": "string"
  },
  "patient": {
    "patientsex": "string (0=Unknown, 1=Male, 2=Female)",
    "patientonsetage": "string (numeric)",
    "patientonsetageunit": "string (800=Decade, 801=Year, 802=Month, 803=Week, 804=Day, 805=Hour)",
    "patientweight": "string (kg)",
    "reaction": [ReactionObject],
    "drug": [DrugObject]
  }
}
```

### Reaction Object
```json
{
  "reactionmeddraversionpt": "string",
  "reactionmeddrapt": "string (MedDRA Preferred Term, e.g., 'NAUSEA')",
  "reactionoutcome": "string (1=Recovered, 2=Recovering, 3=Not recovered, 4=Recovered with sequelae, 5=Fatal, 6=Unknown)"
}
```

### Drug Object
```json
{
  "drugcharacterization": "string (1=Suspect, 2=Concomitant, 3=Interacting)",
  "medicinalproduct": "string (drug name as reported)",
  "drugauthorizationnumb": "string (optional)",
  "drugdosagetext": "string",
  "drugdosageform": "string (optional)",
  "drugindication": "string (optional)",
  "drugadministrationroute": "string (route code, optional)",
  "actiondrug": "string (1=Withdrawn, 2=Dose reduced, 3=Dose increased, 4=Dose not changed, 5=Unknown, 6=Not applicable)",
  "openfda": {
    "application_number": ["string"],
    "brand_name": ["string"],
    "generic_name": ["string"],
    "manufacturer_name": ["string"],
    "product_ndc": ["string"],
    "product_type": ["string"],
    "route": ["string"],
    "substance_name": ["string"],
    "rxcui": ["string"],
    "spl_id": ["string"],
    "spl_set_id": ["string"],
    "package_ndc": ["string"],
    "unii": ["string"],
    "pharm_class_epc": ["string (optional)"],
    "pharm_class_moa": ["string (optional)"],
    "pharm_class_cs": ["string (optional)"],
    "pharm_class_pe": ["string (optional)"]
  }
}
```

---

## Endpoint 2: Drugs@FDA

### Base URL
```
https://api.fda.gov/drug/drugsfda.json
```

### Data Source
Drugs@FDA database. Includes most drug products approved since 1939. Updated daily (Monday–Friday).

### Key Searchable Fields
- `openfda.brand_name` — Brand name
- `openfda.generic_name` — Generic name
- `openfda.manufacturer_name` — Manufacturer
- `openfda.substance_name` — Active ingredient(s)
- `openfda.product_type` — e.g., `HUMAN PRESCRIPTION DRUG`, `HUMAN OTC DRUG`
- `openfda.route` — Route of administration
- `openfda.pharm_class_epc` — Pharmacologic class
- `openfda.application_number` — NDA/ANDA number
- `application_number` — Application number
- `sponsor_name` — Sponsor/company name
- `products.brand_name` — Product brand name
- `products.dosage_form` — Dosage form (e.g., `TABLET`, `CAPSULE`)
- `products.route` — Route
- `products.marketing_status` — e.g., `Prescription`, `Over-the-counter`, `Discontinued`
- `products.active_ingredients.name` — Active ingredient name
- `products.active_ingredients.strength` — Strength
- `submissions.submission_type` — `ORIG` (original), `SUPPL` (supplement)
- `submissions.submission_status` — `AP` (approved), `TA` (tentatively approved)
- `submissions.submission_status_date` — Date (YYYYMMDD)

### Response Schema

```json
{
  "meta": {
    "disclaimer": "string",
    "terms": "string",
    "license": "string",
    "last_updated": "string (YYYY-MM-DD)",
    "results": {
      "skip": "integer",
      "limit": "integer",
      "total": "integer"
    }
  },
  "results": [DrugsFDARecord]
}
```

### Drugs@FDA Record Object

```json
{
  "application_number": "string (e.g., 'NDA021457', 'ANDA075141')",
  "sponsor_name": "string",
  "openfda": {
    "application_number": ["string"],
    "brand_name": ["string"],
    "generic_name": ["string"],
    "manufacturer_name": ["string"],
    "product_ndc": ["string"],
    "product_type": ["string"],
    "route": ["string"],
    "substance_name": ["string"],
    "rxcui": ["string"],
    "spl_id": ["string"],
    "spl_set_id": ["string"],
    "package_ndc": ["string"],
    "unii": ["string"],
    "nui": ["string (optional)"],
    "pharm_class_epc": ["string (optional)"],
    "pharm_class_moa": ["string (optional)"],
    "pharm_class_cs": ["string (optional)"],
    "pharm_class_pe": ["string (optional)"]
  },
  "products": [ProductObject],
  "submissions": [SubmissionObject]
}
```

### Product Object
```json
{
  "product_number": "string",
  "reference_drug": "string (Yes|No)",
  "brand_name": "string",
  "active_ingredients": [
    {
      "name": "string",
      "strength": "string (e.g., '385MG')"
    }
  ],
  "reference_standard": "string (Yes|No)",
  "dosage_form": "string (e.g., TABLET, CAPSULE, INJECTION)",
  "route": "string (e.g., ORAL, INTRAVENOUS)",
  "marketing_status": "string (Prescription|Over-the-counter|Discontinued|None)"
}
```

### Submission Object
```json
{
  "submission_type": "string (ORIG|SUPPL)",
  "submission_number": "string",
  "submission_status": "string (AP=Approved, TA=Tentatively Approved)",
  "submission_status_date": "string (YYYYMMDD)",
  "review_priority": "string (STANDARD|PRIORITY|ORPHAN)",
  "submission_class_code": "string",
  "submission_class_code_description": "string",
  "application_docs": [
    {
      "id": "string",
      "url": "string",
      "date": "string (YYYYMMDD)",
      "type": "string (Review|Label|Letter|Other)"
    }
  ]
}
```

---

## Example API Calls

### Drug Adverse Events

#### Search adverse events for a specific drug
```
GET https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:aspirin&limit=5
```

#### Search serious adverse events for a drug
```
GET https://api.fda.gov/drug/event.json?search=patient.drug.openfda.brand_name:"ADVIL"+AND+serious:1&limit=10
```

#### Search adverse events by date range
```
GET https://api.fda.gov/drug/event.json?search=receivedate:[20230101+TO+20231231]&limit=10
```

#### Search adverse events by pharmacologic class
```
GET https://api.fda.gov/drug/event.json?search=patient.drug.openfda.pharm_class_epc:"nonsteroidal+anti-inflammatory+drug"&limit=5
```

#### Count most common reactions for a drug
```
GET https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:aspirin&count=patient.reaction.reactionmeddrapt.exact
```

#### Search fatal adverse events
```
GET https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:metformin+AND+seriousnessdeath:1&limit=10
```

### Drugs@FDA

#### Search approved drugs by brand name
```
GET https://api.fda.gov/drug/drugsfda.json?search=openfda.brand_name:LIPITOR&limit=5
```

#### Search by active ingredient
```
GET https://api.fda.gov/drug/drugsfda.json?search=openfda.substance_name:METFORMIN&limit=5
```

#### Search by marketing status
```
GET https://api.fda.gov/drug/drugsfda.json?search=products.marketing_status:"Discontinued"&limit=5
```

#### Search by sponsor
```
GET https://api.fda.gov/drug/drugsfda.json?search=sponsor_name:PFIZER&limit=10
```

### Pagination Example
```
# First page
GET https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:aspirin&limit=10&skip=0

# Second page
GET https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:aspirin&limit=10&skip=10
```

---

## Count Query Response

When using the `count` parameter, the response format changes:

```json
{
  "meta": { "...": "..." },
  "results": [
    {
      "term": "DRUG INEFFECTIVE",
      "count": 32584
    },
    {
      "term": "NAUSEA",
      "count": 27541
    }
  ]
}
```

---

## Date Format

- openFDA uses `YYYYMMDD` format (no dashes), e.g., `20240101`
- Date ranges use bracket syntax: `[20200101+TO+20201231]`

---

## Rate Limits

- Without API key: **40 requests per minute**, per IP address
- With API key: **240 requests per minute**
- Obtain a free key at: https://open.fda.gov/apis/authentication/

---

## Notes

1. **No individual field is required** — but at least a `search`, `count`, or no-parameter call is expected
2. **`.exact` suffix** — required on string fields when using `count` to count full phrases instead of individual words
3. **Maximum results** — `limit` max is `1000` per call; `skip` max is `25000`
4. **openFDA fields** — harmonized fields (under `openfda.*`) are added by openFDA to link records across datasets
5. **Causal relationship** — adverse event reports do NOT prove a drug caused a reaction; they are voluntarily reported
6. **URL encoding** — use `+` for spaces in query values; use `"` for exact phrases
7. **Case sensitivity** — field names are case-sensitive and lowercase

---

## Error Responses

The API returns standard HTTP status codes:
- `200 OK` — Successful request
- `400 Bad Request` — Invalid query syntax
- `404 Not Found` — No matches or invalid endpoint
- `429 Too Many Requests` — Rate limit exceeded
- `500 Internal Server Error` — Server error

Error response format:
```json
{
  "error": {
    "code": "string",
    "message": "string"
  }
}
```

---

## References

- Official openFDA Documentation: https://open.fda.gov/apis/
- Drug Adverse Events: https://open.fda.gov/apis/drug/event/
- Drugs@FDA: https://open.fda.gov/apis/drug/drugsfda/
- Query Syntax: https://open.fda.gov/apis/query-syntax/
- Authentication / API Keys: https://open.fda.gov/apis/authentication/
