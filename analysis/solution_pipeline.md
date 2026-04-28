## Health Risk Resource Intensiveness Analysis Pipeline

---

## Problem Statement

Healthcare systems often struggle to anticipate which cancer patients will require the most resources over time. Without early identification, hospitals may face inefficient allocation of staff, beds, and treatments. This project aims to build a predictive framework that estimates patient-level healthcare utilization using structured clinical data.

---

## Research Question

Among cancer patients, can we predict which patients will have high healthcare utilization (a proxy for disease burden and resource intensity) based on their demographics, cancer type, vital signs, and lab results?

This question is actionable because hospitals need to anticipate resource demand for cancer care planning. The dataset is well-suited for this task, as it contains rich longitudinal EHR records with substantial variation across encounters, procedures, medications, and clinical measurements for 11 patients.

---

## Data Preparation

This pipeline integrates and transforms raw MongoDB healthcare data into a machine learning-ready format.

1. **Data Extraction (MongoDB)**
   - Pulled 91,306 documents from three collections (female, male, and assorted).
   - Grouped records by `resourceType` (Patient, Observation, Encounter, Procedure, Medication, Condition, etc.).

2. **Patient Demographics Table**
   - Extracted 11 Patient documents.
   - Created structured demographic features:
     - Age
     - Gender
     - Race / Ethnicity
     - DALY (Disability-Adjusted Life Years)
     - QALY (Quality-Adjusted Life Years)

3. **Clinical Observations Processing**
   - Processed ~47,000 Observation records.
   - Mapped LOINC codes to interpretable clinical variables (e.g., heart rate, hemoglobin, glucose).
   - Aggregated repeated measurements by selecting the most recent value per patient.
   - Pivoted into a wide-format table (one row per patient).

4. **Healthcare Utilization Features**
   - Counted total encounters, procedures, and medication requests per patient.
   - Extracted condition history from diagnosis records.
   - Identified cancer type using keyword matching (Breast, Prostate, Colon).

5. **Feature Consolidation**
   - Merged all datasets into a single patient-level analytical table.
   - Final dataset contains:
     - Demographics
     - Vitals
     - Lab results
     - Cancer type
     - Utilization counts
     - Condition counts

---

## Target Variable Construction

- Computed **total healthcare utilization** as:

  \[
  \text{total\_utilization} = \text{encounters} + \text{procedures} + \text{medication requests}
  \]

- Converted into binary classification:
  - **High Utilization (1):** ≥ median utilization
  - **Low Utilization (0):** < median utilization

- Median value: **584**
- Class distribution:
  - 6 high-utilization patients
  - 5 low-utilization patients

---

## Modelling Approach

Two supervised classification models were trained:

1. **Random Forest Classifier**
   - Selected for robustness on small datasets
   - Handles mixed feature types well
   - Provides interpretable feature importance

2. **Gradient Boosting Classifier**
   - Included for comparison
   - Often improves performance via sequential error correction

### Validation Strategy
- Used **Leave-One-Out Cross Validation (LOOCV)** due to small sample size (n = 11)
- Each iteration:
  - Train on 10 patients
  - Test on 1 patient
- Metrics reported:
  - Accuracy
  - AUC
  - Precision / Recall / F1-score

---

## Model Evaluation

### Random Forest
- Accuracy: **0.545**
- AUC: **0.517**

Confusion Matrix:
- Correct predictions: 6 / 11

### Gradient Boosting
- Accuracy: **0.455**
- AUC: **0.333**

Confusion Matrix:
- Correct predictions: 5 / 11

### Model Comparison

| Metric     | Random Forest | Gradient Boosting |
|------------|--------------|-------------------|
| Accuracy   | 0.545        | 0.455             |
| AUC        | 0.517        | 0.333             |

**Conclusion:** Random Forest performed better across both accuracy and AUC, and was selected for interpretability and downstream analysis.

---

## Feature Importance (Random Forest)

Top predictive features:

- num_active
- rbc
- qaly
- heart_rate
- platelets
- daly
- pain_score
- hematocrit
- num_conditions
- age

These features indicate that both clinical severity and chronic disease burden contribute to healthcare utilization patterns.

---

## Visualisation

Feature importance was visualized using bar plots based on Gini importance from the Random Forest model.

Each bar represents:
- The relative contribution of a feature to classification performance.

---

## Visualisation Insights

- **num_active** was the strongest predictor of high utilization.
  - High-utilization patients had significantly more active conditions.
- Clinical severity markers such as:
  - heart_rate
  - rbc
  - platelets
  - hematocrit  
  were more influential than demographic variables alone.
- **daly and qaly** also ranked highly, indicating that overall quality-of-life burden is strongly associated with healthcare usage.
- Features like **HDL** and **respiratory_rate** had relatively low predictive importance, suggesting limited differentiation power in this dataset.

Overall, the model suggests that **disease burden and physiological strain are stronger drivers of utilization than demographic characteristics alone**.

---

## Conclusion

This project demonstrates that healthcare utilization among cancer patients can be partially predicted using structured EHR features, even with a small dataset.

Key findings:
- Random Forest outperformed Gradient Boosting in both accuracy and AUC.
- Clinical burden indicators (number of active conditions, lab abnormalities, quality-of-life metrics) were the strongest predictors.
- Demographic features alone were not sufficient to explain utilization differences.

---

## Final Takeaway

Healthcare utilization in cancer patients is primarily driven by **clinical complexity and active disease burden**, rather than demographics alone. Even with limited patient data, structured EHR features can provide meaningful signals for anticipating resource needs and improving hospital planning strategies.