# DS 4320 Project 2: Predicting Healthcare Utilization in Cancer Patients

## Executive Summary
This project operates in the healthcare analytics domain with a focus on oncology and clinical data modeling using a document-based MongoDB architecture. Leveraging longitudinal synthetic cancer patient records from the mCODE Synthea dataset, the project integrates complex, nested healthcare data—including patient demographics, clinical observations, lab results, and encounter histories—into a unified analytical pipeline. The primary objective is to identify patterns in patient health trajectories and develop a predictive model that classifies patients into high and low healthcare resource utilisation categories, serving as a proxy for disease burden and clinical resource intensity.

---

## Project Information

- **Name:** Anaya Nath  
- **NetID:** dtv9vd 
- **DOI:** [10.5281/zenodo.19865590](https://doi.org/10.5281/zenodo.19865590)
- **License:** https://github.com/Anaya666/oncology_dp2/blob/main/docs/LICENSE


**Links:**
- Press Release: https://github.com/Anaya666/oncology_dp2/blob/main/docs/press_release.md  
- Data: https://confluence.hl7.org/spaces/COD/pages/80119851/mCODE+Test+Data 
- Pipeline: https://github.com/Anaya666/oncology_dp2/tree/main/analysis/solution_pipeline.ipynb  
---

## Problem Definition

### Initial General Problem
Understanding what drives differences in healthcare resource utilisation among cancer patients. 

### Refined Specific Problem
Identify which clinical features (demographics, vitals, lab results, and encounter history) most strongly contribute to high vs low healthcare resource utilization among cancer patients. 

### Rationale
Cancer patients exhibit highly variable healthcare needs due to differences in disease severity, comorbid conditions, and physiological responses, making it important to move beyond a general classification of high versus low resource utilisation by patients toward understanding which specific clinical factors drive healthcare resource utilization. This provides interpretable insights into the most influential features, supporting more informed clinical decision-making and resource planning in oncology care.

### Motivation
My motivation for this project comes from a personal experience: a family member was diagnosed with stage 1 breast cancer a few years ago and spent a significant amount of time in and out of the hospital, continuing with regular follow-up check-ups even after initial treatment. Seeing how much clinical data, monitoring, and ongoing care is involved for a single patient made me realize how complex and resource-intensive cancer care can be at the individual level. This led me to explore how hospitals can better understand and anticipate resource utilization across patients using data, with the goal of improving care planning and ensuring resources are allocated where they can have the greatest impact.

---

## Domain Exposition
This project operates in the domain of clinical informatics and oncology, where  large volumes of patient health records are used to understand disease progression, treatment patterns, and healthcare resource consumption. Cancer care is among the  most data-intensive areas of medicine: a single patient can generate thousands of clinical records spanning lab results, medications, procedures, and encounters over the course of their treatment. The emergence of standardized healthcare data formats such as FHIR (Fast Healthcare Interoperability Resources) and domain-specific standards like mCODE (Minimal Common Oncology Data Elements) has made it possible to structure and analyze this data at scale. Document databases such as MongoDB are particularly well-suited for this domain because healthcare records are inherently nested, variable in structure, and patient-centric:  properties that align naturally with the document model. By combining clinical data engineering with machine learning,  this project sits at the intersection of health informatics, data science, and oncology research, with the practical goal of identifying which patients are most 
likely to require intensive healthcare resources.

### Key Metrics and Jargon

## Key Metrics and Jargon

| Term | Definition | Relevance to Project |
|------|-----------|---------------------|
| Patient | An individual receiving medical care, represented as a document in the dataset | Core unit of analysis; each document corresponds to one patient |
| Encounter | A recorded interaction between a patient and healthcare provider | Used to track patient history over time |
| Observation | Clinical measurements such as vitals or lab results (e.g., blood pressure, hemoglobin) | Key features used to assess patient health and risk |
| Diagnostic Report | Summary of clinical tests and results | Provides aggregated medical insights for each patient |
| Longitudinal Data | Data collected over time for the same patient | Enables tracking changes in health status |
| Risk Score | A computed value indicating a patient’s health risk level | Target variable for analysis and modeling |
| Vital Signs | Basic health indicators (e.g., heart rate, blood pressure) | Important predictors of patient condition |
| Lab Results | Quantitative test results (e.g., leukocyte count, hemoglobin levels) | Used to detect abnormalities and health trends |
| Document Database | A database model (e.g., MongoDB) that stores data as JSON-like documents | Matches the nested structure of healthcare data |
| FHIR (Fast Healthcare Interoperability Resources) | Standard for structuring healthcare data | Defines the schema used in the dataset |
| mCODE | Minimal Common Oncology Data Elements standard for cancer data | Ensures consistent cancer-related data representation |
| Feature Engineering | Process of transforming raw data into usable inputs for models | Used to create risk indicators from observations |
| Data Pipeline | Sequence of steps for data processing, analysis, and modeling | Organizes the workflow of the project |
| Aggregation | Combining data across records (e.g., average lab values) | Used to summarize patient-level insights |
| Model Prediction | Output of a machine learning model | Used to estimate patient health risk |

---

## Total Utilisation = patient encounter count + procedure count + medication request count


---

### Background Reading Summary


| Article Title | Brief Description | Link |
|--------------|-----------------|------|
| Healthcare Data & Innovation | Healthcare organizations use large volumes of data, analytics, and AI to improve patient outcomes, optimize resources, and enable data-driven decision-making. | [View Article](https://drive.google.com/file/d/1PY0sJxQo_GkLbHlr5hqYRFt1-LQNpZ95/view?usp=sharing) |
| mCODE & Cancer Data Standards | mCODE standardizes cancer data using FHIR, enabling interoperable, structured, and shareable data for improved care coordination and research insights. | [View Article](https://drive.google.com/file/d/1YFKwIYNl0VG3kc1uIkMjpgYqA1JYqBzN/view?usp=sharing) |
| AI in Oncology | AI supports treatment decisions and personalization in cancer care but introduces ethical challenges such as bias, privacy risks, and lack of transparency. | [View Article](https://drive.google.com/file/d/1cHBUUziGVISF6ydonppX4IQKqhaJM-2q/view?usp=sharing) |
| ML for Cancer Risk Prediction | Machine learning models analyze symptoms and clinical data to predict cancer risk, though current models require better validation and bias reduction. | [View Article](https://drive.google.com/file/d/1HDCcwc40nWhFdf45yy8X3nicnDRYuH8T/view?usp=sharing) |
| Electronic Health Records (EHRs) | EHRs store comprehensive patient histories and use AI to improve decision-making, streamline workflows, and enhance care coordination. | [View Article](https://drive.google.com/file/d/166oxPXlhzCADu2wnr8iXXEk5N56zTDOU/view?usp=sharing) |
---

## Data Creation

### Data Acquisition
The dataset used in this project consists of synthetic longitudinal oncology patient records obtained from the official Synthea downloads page at synthea.mitre.org/downloads. Specifically, the Breast Cancer FHIR and mCODE dataset was used, which was produced on March 6, 2020 using Synthea™ and is conformant to the mCODE STU1 (Minimal Common Oncology Data Elements) standard. The dataset was downloaded as a compressed ZIP file (82 MB) from the MITRE Box repository and contains approximately 200 lifetime/longitudinal patient records spanning three categories: 178 female breast cancer patients, 13 male breast cancer patients, and 5 assorted other cancer patients (lung, colorectal, and prostate). Each patient record is stored as an individual JSON file in FHIR Bundle format, where each file contains a list of clinical resource entries such as Patient demographics, Encounters, Observations, Conditions, and Diagnostic Reports.

After downloading, the dataset was uploaded to Google Drive and loaded into MongoDB Atlas using PyMongo in Google Colab. Before loading, the structure of the JSON files was explored to understand their format. Each file was found to have three top-level keys, resourceType, type, and entry, where the entry key contained the entire clinical record as a large list. A single patient file was found to contain up to 2,740 individual entries, with the entry list alone reaching over 4 million characters. Because each raw JSON file was too large to insert as a single MongoDB document (exceeding the 16MB BSON limit), the data was split at the entry level,each entry within a patient file was extracted and inserted as its own individual document in MongoDB. This splitting strategy allowed the data to fit within MongoDB's document size constraints while preserving all clinical information. To ensure a balanced and unbiased dataset, the female and male collections were capped at approximately 12,500–13,500 documents each, while all assorted patient files were fully loaded. The final dataset contains over 37,000 documents across three collections, female, male, and assorted, within a database named longitudinal_oncology, well exceeding the required threshold for analysis.

---

### Code Table

## Code Files Overview

| File | Description | Link |
|------|-------------|------|
| `data_creation.ipynb` | Mounts Google Drive, explores FHIR JSON structure by inspecting top level keys and entry sizes, splits each patient bundle into individual entry-level documents, and loads female/male/assorted collections into MongoDB Atlas (`longitudinal_oncology`) with female and male capped at ~12,500 documents each for a balanced dataset | [GitHub link] https://github.com/Anaya666/oncology_dp2/blob/main/data_creation.ipynb) |

---

### Bias Identification
The primary source of bias in the data collection process was selection bias introduced by the file loading order. Since the dataset had to be capped at approximately 12,500 documents per gender group due to MongoDB Atlas storage constraints, the initial loading approach selected files in alphabetical order by filename, meaning only patients whose names appeared earliest alphabetically had a chance of being included. This systematically excluded a large portion of patients, making the sample unrepresentative of the full dataset. Additionally, the dataset itself is synthetically generated using the Synthea framework, which means it reflects the demographic assumptions and population distributions built into the simulation model rather than real-world patient populations. This could introduce bias in age distributions, race, ethnicity, and geographic representation that may not reflect the true diversity of cancer patients.

---

### Bias Mitigation
To mitigate selection bias introduced by alphabetical file ordering, the patient files were randomly shuffled using a fixed random seed (42) before loading, ensuring every patient had an equal probability of being selected regardless of their name. The random seed ensures the process is reproducible, meaning the same random sample can be regenerated if needed. To address gender bias, the female and male collections were deliberately capped at similar document counts (~12,500–13,500 each), ensuring neither gender dominates the dataset during analysis and modeling. For the synthetic data bias, the known demographic distributions from the Synthea simulation can be quantified and accounted for during the modeling phase by examining feature distributions, checking for class imbalances, and applying techniques such as stratified sampling or reweighting if certain demographic groups are over or underrepresented.

---

### Decision Rationale
Several critical decisions were made during the data creation process that involved judgment calls and introduced or mitigated uncertainty.

1. Splitting at the entry level: The most significant technical decision was splitting each FHIR Bundle JSON file into individual entry-level documents rather than storing each patient file as a single document. This was necessary because several patient files exceeded MongoDB's 16MB BSON document size limit (some reaching up to 56MB). While this splitting strategy solved the size constraint, it introduces uncertainty by separating related clinical records that originally belonged to the same patient, for example, a patient's Observation and their Encounter are now stored as separate documents. To mitigate this, each document retains a source_file field that links it back to its original patient file, allowing records to be regrouped by patient during analysis.

2. Capping at 12,500 documents per gender: The decision to cap female and male collections at approximately 12,500–13,500 documents was driven by the 512MB storage limit of the free MongoDB Atlas tier. This cap introduces uncertainty because the selected subset may not fully represent the diversity of clinical histories present in the complete dataset. This was partially mitigated by randomly shuffling files before loading using a fixed random seed (42), ensuring the selection was not systematically biased toward any particular group of patients.

3. Excluding male collection initially: The male collection was initially excluded entirely due to storage constraints before the female collection was cleared and reloaded. This was a temporary judgment call that was corrected by deliberately balancing both gender groups, reducing the risk of a gender-biased dataset that could produce unfair predictions in the downstream health risk model.

---


## Metadata

### Implicit Schema
Since MongoDB does not enforce a strict schema, the following guidelines define the expected document structure for the `longitudinal_oncology` database. All documents across the female, male, and assorted collections share a common base structure inherited from the FHIR standard, with additional fields varying by `resourceType`.

Guidelines for ALL Documents:
All documents regardless of `resourceType` must follow these guidelines:


- `resourceType` — **required** — string identifying the type of clinical record
(e.g. `"Patient"`, `"Observation"`, `"Encounter"`)
- `id` — **required** — unique UUID string identifying the resource
- `source_file` — **required** — string linking the document back to its original
patient JSON file
- `gender_group` — **required** — string indicating which collection the patient
belongs to (`"female"`, `"male"`, or `"assorted"`)
- `meta.profile` — **recommended** — list of FHIR/mCODE profile URLs confirming
conformance to mCODE STU1
- `text` — **optional** — human readable narrative summary of the document

- Guidelines for `Patient` Documents
- `birthDate` — **required** — string in `YYYY-MM-DD` format
- `gender` — **required** — string (`"female"` or `"male"`)
- `name` — **required** — list of name objects containing `family`, `given`,
`prefix`, and `use`
- `address` — **recommended** — list containing city, state, country, and
geolocation coordinates
- `identifier` — **required** — list of identifiers including Medical Record
Number, SSN, Driver's License, and Passport Number
- `communication` — **recommended** — list containing the patient's preferred
language
- `maritalStatus` — **optional** — coded value for marital status
- `extension` — **recommended** — list containing race, ethnicity, birth sex,
birthplace, DALY, and QALY values

- Guidelines for All Other resourceTypes
(`Observation`, `Encounter`, `Condition`, `Procedure`, etc.)
- `subject` — **required** — reference linking back to the parent Patient
document via UUID
- `status` — **required** — string indicating the status of the resource
(e.g. `"final"`, `"finished"`)
- `code` — **required** — coded value using standard medical coding systems
(SNOMED, LOINC, etc.)
- `effectiveDateTime` or `period` — **recommended** — timestamp or date range
of the clinical event

- Valid resourceType Values
The 19 valid `resourceType` values in this database are:

`Patient`, `Observation`, `Encounter`, `Condition`, `Procedure`,
`MedicationRequest`, `MedicationStatement`, `MedicationAdministration`,
`Medication`, `CarePlan`, `CareTeam`, `DiagnosticReport`, `DocumentReference`,
`ImagingStudy`, `Immunization`, `Location`, `Organization`, `Practitioner`,
`PractitionerRole`

---

### Data Summary
Database Overview

| Collection | Documents | Resource Types | Description |
|------------|-----------|----------------|-------------|
| female | 41,826 | 19 | Female breast cancer patient records |
| male | 38,573 | 17 | Male breast cancer patient records |
| assorted | 10,907 | 17 | Assorted other cancer patient records (lung, colorectal, prostate) |
| **Total** | **91,306** | **19** | **Full longitudinal oncology dataset** |

Resource Type Breakdown (Female Collection)

| Resource Type | Count | Description |
|---------------|-------|-------------|
| Observation | 21,325 | Clinical measurements, vitals, lab results |
| DiagnosticReport | 4,945 | Summary of clinical tests and results |
| Procedure | 4,236 | Medical procedures performed |
| Encounter | 3,807 | Patient-provider interactions |
| DocumentReference | 3,807 | References to clinical documents |
| MedicationRequest | 2,913 | Medication prescriptions |
| Immunization | 303 | Vaccination records |
| Condition | 197 | Diagnosed medical conditions |
| CarePlan | 77 | Patient care plans |
| CareTeam | 77 | Care team members |
| MedicationStatement | 28 | Medication usage statements |
| Practitioner | 22 | Healthcare providers |
| Location | 22 | Clinical locations |
| Organization | 22 | Healthcare organizations |
| PractitionerRole | 22 | Practitioner roles |
| Medication | 6 | Medication details |
| ImagingStudy | 6 | Medical imaging records |
| MedicationAdministration | 6 | Medication administration records |
| Patient | 5 | Core patient demographic records |

Key Observations
- Observations are by far the most common resource type (~51% of female
collection), reflecting the longitudinal nature of the dataset with repeated
clinical measurements over time
- Patient documents are the least common (only 5 per collection) since each
file represents one patient:all other documents are clinical events tied to
those patients
- The resource type distribution is consistent across all three collections

---

### Data Dictionary 
- Universal Fields (All Documents)

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `_id` | ObjectId | MongoDB auto-generated unique document identifier | `ObjectId('69e5164871928ecc8ecd23ac')` |
| `resourceType` | String | FHIR resource type identifying the clinical record type | `"Patient"` |
| `id` | String | Unique UUID identifying the resource | `"e026fc6f-e39a-4c89-91bb-e8b81bb46c7c"` |
| `source_file` | String | Original patient JSON filename the document was extracted from | `"Sofia418_Mata817_e026fc6f.json"` |
| `gender_group` | String | Collection the document belongs to | `"female"` |
| `meta.profile` | Array | List of FHIR/mCODE profile URLs for conformance | `["http://hl7.org/fhir/us/mcode/..."]` |
| `text` | Object | Human readable narrative summary of the document | `{"status": "generated", "div": "..."}` |

- Patient Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `birthDate` | String | Patient date of birth in YYYY-MM-DD format | `"1956-05-30"` |
| `gender` | String | Patient biological sex | `"female"` |
| `name` | Array | List of name objects with family, given, prefix, use | `[{"family": "Mata817", "given": ["Sofia418"]}]` |
| `address` | Array | Patient address including city, state, country, geolocation | `[{"city": "Braintree", "state": "MA", "country": "US"}]` |
| `telecom` | Array | Contact information including phone number | `[{"system": "phone", "value": "555-754-5683"}]` |
| `maritalStatus` | Object | Coded marital status value | `{"text": "M"}` |
| `multipleBirthBoolean` | Boolean | Whether patient is a multiple birth | `False` |
| `communication` | Array | Patient's preferred language | `[{"language": {"text": "Spanish"}}]` |
| `identifier` | Array | List of IDs including MRN, SSN, Driver's License, Passport | `[{"system": "...", "value": "999-44-4162"}]` |
| `extension` | Array | Extended attributes including race, ethnicity, DALY, QALY | `[{"url": "us-core-race", "valueString": "White"}]` |

- Observation Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `status` | String | Status of the observation | `"final"` |
| `category` | Array | Category of observation (e.g. vital-signs, laboratory) | `[{"coding": [{"code": "vital-signs"}]}]` |
| `code` | Object | LOINC coded description of what was measured | `{"text": "Body Height", "coding": [{"code": "8302-2"}]}` |
| `subject` | Object | Reference UUID linking back to parent Patient | `{"reference": "urn:uuid:e026fc6f..."}` |
| `encounter` | Object | Reference to the encounter during which observation was made | `{"reference": "urn:uuid:97dd4c4b..."}` |
| `effectiveDateTime` | String | Timestamp when observation was recorded | `"1956-05-30T20:28:23-04:00"` |
| `issued` | String | Timestamp when observation was issued | `"1956-05-30T20:28:23.182-04:00"` |
| `valueQuantity` | Object | Numeric measurement value with unit | `{"value": 50.2, "unit": "cm"}` |

- Encounter Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `status` | String | Status of the encounter | `"finished"` |
| `class` | Object | Classification of encounter type | `{"code": "AMB"}` |
| `type` | Array | SNOMED coded type of encounter | `[{"text": "Well child visit (procedure)"}]` |
| `subject` | Object | Reference UUID linking back to parent Patient | `{"reference": "urn:uuid:e026fc6f..."}` |
| `participant` | Array | Healthcare providers involved in encounter | `[{"individual": {"display": "Dr. Darrick836 Lemke654"}}]` |
| `period` | Object | Start and end datetime of encounter | `{"start": "1956-05-30T20:28:23", "end": "1956-05-30T20:43:23"}` |
| `location` | Array | Clinical location where encounter occurred | `[{"location": {"display": "PCP2779"}}]` |
| `serviceProvider` | Object | Organization providing the service | `{"display": "PCP2779"}` |

- Condition Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `clinicalStatus` | Object | Current clinical status of condition | `{"coding": [{"code": "resolved"}]}` |
| `verificationStatus` | Object | Verification status of condition | `{"coding": [{"code": "confirmed"}]}` |
| `category` | Array | Category of condition | `[{"coding": [{"code": "encounter-diagnosis"}]}]` |
| `code` | Object | SNOMED coded diagnosis | `{"text": "Otitis media", "coding": [{"code": "65363002"}]}` |
| `onsetDateTime` | String | Timestamp when condition began | `"1958-09-17T20:28:23-04:00"` |
| `abatementDateTime` | String | Timestamp when condition resolved | `"1958-11-05T19:28:23-05:00"` |
| `recordedDate` | String | Timestamp when condition was recorded | `"1958-09-17T20:28:23-04:00"` |

- Procedure Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `status` | String | Status of the procedure | `"completed"` |
| `code` | Object | SNOMED coded procedure description | `{"text": "Medication Reconciliation (procedure)"}` |
| `subject` | Object | Reference UUID linking back to parent Patient | `{"reference": "urn:uuid:e026fc6f..."}` |
| `performedPeriod` | Object | Start and end datetime of procedure | `{"start": "1956-09-05T20:28:23", "end": "1956-09-05T20:43:23"}` |
| `location` | Object | Location where procedure was performed | `{"display": "PCP2779"}` |

-  DiagnosticReport Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `status` | String | Status of the diagnostic report | `"final"` |
| `category` | Array | Category of report | `[{"coding": [{"code": "LAB", "display": "Laboratory"}]}]` |
| `code` | Object | LOINC coded report type | `{"text": "Complete blood count panel"}` |
| `effectiveDateTime` | String | Timestamp when report was effective | `"1956-05-30T20:28:23-04:00"` |
| `issued` | String | Timestamp when report was issued | `"1956-05-30T20:28:23.182-04:00"` |
| `performer` | Array | Provider who performed the report | `[{"display": "PCP2779"}]` |
| `result` | Array | List of references to Observation results | `[{"display": "Leukocytes [#/volume] in Blood"}]` |

-  MedicationRequest Fields

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `status` | String | Status of the medication request | `"completed"` |
| `intent` | String | Intent of the medication request | `"order"` |
| `medicationCodeableConcept` | Object | RxNorm coded medication name and dosage | `{"text": "Penicillin G 375 MG/ML Injectable Solution"}` |
| `authoredOn` | String | Timestamp when request was authored | `"1958-09-17T20:28:23-04:00"` |
| `requester` | Object | Provider who requested the medication | `{"display": "Dr. Billie243 Ferry570"}` |

### Uncertainty Quantification
All statistics are computed from the `female` collection across all
`Observation` documents with a `valueQuantity.value` field. Standard deviation
is used as the primary measure of uncertainty — a higher standard deviation
indicates greater variability and uncertainty in that feature.

| Feature | Unit | Count | Mean | Std Dev | Min | Max | Uncertainty Notes |
|---------|------|-------|------|---------|-----|-----|-------------------|
| Pain severity (0-10 verbal numeric rating) | {score} | 2,643 | 3.61 | 1.45 | 0.00 | 8.00 | Moderate uncertainty — subjective self-reported score prone to patient bias |
| Weight difference (pre/post dialysis) | kg | 2,139 | 3.00 | 1.16 | 1.00 | 5.00 | Low-moderate uncertainty — consistent range suggests reliable measurement |
| Potassium (Moles/volume in Serum) | mmol/L | 533 | 4.46 | 0.44 | 3.71 | 5.20 | Low uncertainty — tight range, reliable lab measurement |
| Sodium (Moles/volume in Blood) | mmol/L | 533 | 140.00 | 2.29 | 136.01 | 143.99 | Low uncertainty — very stable physiological range |
| Creatinine (Mass/volume in Serum) | mg/dL | 533 | 4.80 | 6.34 | 0.64 | 49.38 | **High uncertainty** — very high std dev relative to mean, likely reflects severe kidney disease variability |
| Carbon dioxide total (Moles/volume in Serum) | mmol/L | 533 | 24.51 | 2.66 | 20.04 | 28.99 | Low uncertainty — stable physiological range |
| Urea nitrogen (Mass/volume in Serum) | mg/dL | 533 | 13.88 | 3.85 | 7.00 | 19.98 | Low-moderate uncertainty — normal physiological variation |
| Glucose (Mass/volume in Blood) | mg/dL | 533 | 111.12 | 9.60 | 65.51 | 124.99 | Moderate uncertainty — reflects variation in fasting state and diabetes status |
| Calcium (Mass/volume in Serum) | mg/dL | 533 | 9.33 | 0.50 | 8.50 | 10.20 | Low uncertainty — tightly regulated physiological range |
| Chloride (Moles/volume in Blood) | mmol/L | 533 | 106.09 | 2.83 | 101.01 | 110.98 | Low uncertainty — stable electrolyte range |
| Glomerular Filtration Rate (MDRD) | mL/min/1.73m² | 508 | 24.85 | 27.93 | 1.15 | 160.78 | **High uncertainty** — extremely high std dev reflects wide range of kidney function across patients |
| Body Weight | kg | 493 | 59.92 | 23.58 | 3.70 | 86.70 | **High uncertainty** — wide range reflects longitudinal data from birth to adulthood |
| Body Height | cm | 493 | 143.98 | 29.14 | 48.70 | 166.20 | **High uncertainty** — wide range reflects longitudinal data from birth to adulthood |
| Respiratory Rate | /min | 493 | 14.03 | 1.15 | 12.00 | 16.00 | Low uncertainty — tightly regulated normal range |
| Heart Rate | /min | 493 | 79.32 | 11.70 | 60.00 | 100.00 | Low-moderate uncertainty — normal physiological variation |
| Body Mass Index (BMI) | kg/m² | 447 | 27.40 | 4.74 | 15.00 | 36.66 | Moderate uncertainty — reflects variation in patient age and health status |
| Cholesterol (Mass/volume in Serum) | mg/dL | 317 | 211.26 | 18.83 | 150.68 | 238.96 | Low-moderate uncertainty — normal population variation |
| Triglycerides (Mass/volume in Serum) | mg/dL | 317 | 167.42 | 24.74 | 100.26 | 383.21 | Moderate uncertainty — wide max suggests some outlier patients |
| Cholesterol LDL (Direct assay) | mg/dL | 317 | 127.54 | 20.44 | 52.55 | 194.28 | Moderate uncertainty — reflects variation in diet and treatment |
| Cholesterol HDL (Mass/volume in Serum) | mg/dL | 317 | 52.23 | 8.92 | 32.68 | 79.86 | Low-moderate uncertainty — normal population variation |

---

## Press Release
https://github.com/Anaya666/oncology_dp2/blob/main/docs/press_release.md
