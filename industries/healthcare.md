# Healthcare

> **TL;DR** — Healthcare data is the most heterogeneous, most regulated, and most semantically dense data a working data scientist will touch. A single patient produces structured codes (ICD, CPT), free-form notes, numeric lab results in non-standard units, time-series vitals, images, genomics, and increasingly wearable streams — each with its own coding system, units, and missingness patterns. The combination of high stakes (patient outcomes), strict regulation (HIPAA, GDPR, FDA), and unstandardized vendor data makes "just load the CSV" almost never work.

## 1. Common data types

Healthcare data spans every category in [`../data_types/`](../data_types/), often in the same patient record.

| Data type | Where it appears | Notes |
|---|---|---|
| **Categorical (high-cardinality coded)** | Diagnoses (ICD-10/11), procedures (CPT, HCPCS), drugs (RxNorm, NDC), labs (LOINC), terms (SNOMED CT) | Tens of thousands of distinct codes; hierarchical; codes change over time |
| **Numerical (with units!)** | Lab results, vitals, dosages | Mixed units: mg/dL vs. mmol/L, kg vs. lb, °C vs. °F |
| **Time series** | Vitals (HR, BP, SpO₂, RR, temp), ICU monitors, ECG, EEG, continuous glucose | Sample rates from 1/min to 1 kHz |
| **Text** | Clinical notes (H&P, progress, discharge summary), radiology / pathology reports | Mixed structured + free-form; abbreviations, negation, dictation errors |
| **Image** | DICOM (radiology, cardiology), pathology slides (WSI), photos | DICOM carries PHI in headers; pathology files reach GB per slide |
| **Waveform** | ECG, EEG, EMG, fetal monitoring | Streamed, dense, requires DSP |
| **Genomic / -omic** | VCF (variants), BAM/CRAM (alignments), expression matrices | Specialized formats; high-dimensional |
| **Graph** | Care pathways, drug-drug interactions, knowledge graphs (UMLS) | Often as ontologies |
| **Geospatial** | Patient address, hospital location, exposure / environmental data | PHI implications |
| **Wearable / RPM** | Continuous HR, steps, sleep, glucose | Quality variability; consent and provenance |

## 2. Common sources

| Source | What it contains |
|---|---|
| **EHR (Epic, Cerner / Oracle Health, Athenahealth, eClinicalWorks, MEDITECH, Allscripts)** | Encounters, orders, results, meds, notes |
| **Claims (CMS Medicare, Medicaid MAX/T-MSIS, commercial payers, IQVIA, Symphony Health)** | Billing-level diagnoses, procedures, dispensed meds |
| **Registries** | Disease-specific (cancer SEER, NCDR cardiovascular, trauma) |
| **Clinical trial CRFs / EDC** | Medidata Rave, Veeva CDMS, OpenClinica |
| **Biobanks** | UK Biobank, All of Us, FinnGen — often gated |
| **HIE (Health Information Exchanges)** | Multi-payer / multi-provider aggregations |
| **Wearables / RPM** | Apple Health, Fitbit, Garmin, Dexcom CGM, Omron |
| **Medical devices / monitors** | Philips, GE, Mindray; via HL7 v2 or proprietary |
| **Imaging PACS** | Radiology / cardiology image stores |
| **Lab vendors** | Quest, LabCorp, hospital labs |
| **Public** | CDC WONDER, NHANES, BRFSS, NHIS |

## 3. Standard schemas and exchange formats

| Standard | Used for | Notes |
|---|---|---|
| **HL7 v2** | Message-based EHR integration | Pipe-delimited, ubiquitous, idiosyncratic |
| **FHIR** | Modern REST API for clinical data | JSON / XML resources; STU3 / R4 / R5 |
| **C-CDA** | Document exchange (discharge summaries, care plans) | XML, embedded structured + narrative |
| **DICOM** | Medical imaging | Pixel data + metadata in same file |
| **NCPDP** | Pharmacy claims | |
| **X12 837/835** | Claims and remittance | EDI |
| **OMOP CDM** (OHDSI) | Research data model | Normalizes disparate EHRs to a common schema |
| **Sentinel CDM / PCORnet CDM** | Comparative effectiveness research | |
| **VCF, BAM/CRAM, FASTQ** | Genomics |  |
| **i2b2 / Sentinel CDM** | Research data warehouse models | |

### Code systems

| System | What it codes | Stewards |
|---|---|---|
| ICD-10-CM / ICD-11 | Diagnoses | WHO / NCHS |
| CPT, HCPCS | Procedures (US) | AMA / CMS |
| SNOMED CT | Clinical terminology (broadest) | SNOMED International |
| LOINC | Laboratory & clinical observations | Regenstrief |
| RxNorm | Drugs (US) | NLM |
| ATC | Drug classification (intl.) | WHO |
| NDC | Drug product (US) | FDA |
| UMLS | Mega-thesaurus mapping the above | NLM |

Mapping between systems is a constant data-science task; tools include the NLM UMLS API, RxNav, and OHDSI Athena.

## 4. Cleaning particulars

Beyond the generic cleaning in [`../data_cleaning/`](../data_cleaning/):

### 4.1 Unit normalization

Same lab name, different units:

```
Glucose: 110 mg/dL ≈ 6.1 mmol/L
Hemoglobin: 14 g/dL ≈ 140 g/L
Creatinine: 1.0 mg/dL ≈ 88.4 μmol/L
```

LOINC codes are unit-aware *in theory*, but in practice the same LOINC arrives with different units from different labs. Always check and standardize.

### 4.2 Code-system mapping and versions

ICD-9 → ICD-10 (US: October 2015), then ICD-10 → ICD-11 (rolling). A cohort spanning the transition needs crosswalks (GEMs from CMS). Mappings are not 1:1 — document the choice.

### 4.3 Cohort construction (the work nobody warns you about)

"Patients with diabetes" can mean:

- ICD code for diabetes ever recorded.
- ≥ 2 ICD codes ≥ 30 days apart.
- HbA1c ≥ 6.5% in two consecutive measurements.
- Prescription of an antidiabetic drug.
- Combinations thereof.

Each definition produces a *different* cohort with different incidence. Always pre-register the cohort definition.

### 4.4 Index date and look-back / look-forward windows

Every analysis needs an "index date" per patient and well-defined windows around it. Common bugs:

- Using future information (lab drawn after index) as a baseline covariate.
- Including patients who weren't observable for the full look-back period (immortal-time bias).
- Right-censoring without modeling it.

### 4.5 Note de-identification

Free-text notes contain names, dates, MRNs, addresses. Tools: Philter, MIT Deid, AWS Comprehend Medical, Microsoft Presidio + custom recognizers. Validate with manual review — recall < 100% means residual PHI.

### 4.6 DICOM PHI

DICOM headers carry patient name, DOB, MRN, accession number, sometimes referring physician. Strip with `pydicom` deid or DicomCleaner — but pixel data can also contain burned-in text (timestamps, patient name overlays), which requires OCR-based redaction.

### 4.7 Death and discharge

"Did this patient die?" Sources disagree: EHR death flag, claims dates, Social Security Death Master File (US, restricted), National Death Index. Mortality undercount is real in EHR-only studies.

### 4.8 Patient identity

Across sources, the same patient may have different IDs (MRN, NPI patient ID, payer ID, study ID). Master Patient Index / probabilistic record linkage (see [`../data_cleaning/duplicates.md`](../data_cleaning/duplicates.md)) is its own subfield.

## 5. Standard analyses

| Analysis | Typical models / methods |
|---|---|
| **Risk stratification** (readmission, mortality, sepsis, deterioration) | Logistic regression, GBM, deep models on time series |
| **Phenotyping** | Rule-based, weak supervision, contrastive learning on EHRs |
| **Survival analysis** | Kaplan-Meier, Cox PH, AFT, deep survival (DeepHit, RNN-SURV) |
| **Causal inference (real-world evidence)** | Propensity scores, inverse-probability weighting, target trial emulation |
| **Imaging diagnosis** | CNNs / ViTs for classification, U-Net for segmentation |
| **Clinical NLP** | NER, relation extraction, negation/uncertainty (NegEx, BioBERT, ClinicalBERT, Med-PaLM, GPT-4) |
| **Cost / utilization** | GLM with gamma family, two-part models, hierarchical Bayes |
| **Drug discovery / repurposing** | GNNs on molecule graphs, similarity-based scoring |
| **Fraud / billing anomalies** | Outlier detection on provider patterns |
| **Genomic association** | GWAS, polygenic risk scores |
| **Operations** | Census forecasting, OR scheduling, no-show prediction |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Time-to-event | **Kaplan-Meier** survival curves with at-risk table |
| Subgroup effects | **Forest plot** |
| Individual patient trajectory | **Swimmer plot** (per-patient timeline) |
| Population health over time | **Funnel** (incidence → diagnosis → treatment → outcome) |
| Encounter pattern | **Calendar heatmap**, **patient timeline (Gantt-like)** |
| Lab trajectories | **Line per patient with population band** |
| Risk model calibration | **Calibration plot**, **decision curve** |
| Cohort flow | **CONSORT diagram** (always, for any published cohort) |
| Imaging overlays | Mask / annotation overlay, Grad-CAM |
| Geographic disease | **Choropleth** of incidence (equal-area projection) |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **HIPAA (US)** | Protected Health Information (PHI); Privacy and Security Rules; 18 identifiers |
| **HITECH** | Breach notification; meaningful use |
| **21 CFR Part 11** | FDA electronic records / signatures |
| **FDA SaMD / 510(k) / De Novo / PMA** | Software as a Medical Device approvals |
| **EU GDPR + national health laws** | Special-category data; national variations |
| **EU MDR / IVDR** | Medical device regulation |
| **Common Rule (45 CFR 46)** | Human subjects research; IRB |
| **GINA (US)** | Genetic information non-discrimination |
| **State laws** | CMIA (California), Texas Medical Privacy Act, etc. |

### De-identification under HIPAA

Two safe harbors:

- **Safe Harbor** — remove 18 specified identifiers (names, dates more granular than year, ZIP > first 3 digits, etc.).
- **Expert Determination** — a qualified statistician certifies very small re-identification risk.

Synthetic data and differential privacy are increasingly considered when sharing.

### Bias and equity

Algorithmic bias in clinical models is well-documented: race-adjusted eGFR (now revised), pulse oximetry under-reading on dark skin, race as a coefficient in cardiology / OB risk scores. Audit by subgroup *always*. AHRQ, FDA, and ONC have begun issuing guidance.

## 8. Public datasets

| Dataset | What |
|---|---|
| **MIMIC-III, MIMIC-IV** | Beth Israel ICU, requires CITI training |
| **eICU Collaborative** | Multi-center ICU |
| **NHANES** | US National Health and Nutrition Examination Survey |
| **BRFSS** | Behavioral Risk Factor Surveillance System |
| **CDC WONDER** | Mortality, natality, disease |
| **SEER** | US cancer registries |
| **UK Biobank, All of Us** | Large cohorts, gated |
| **PhysioNet** | Hosts MIMIC, plus many ECG / EEG / waveform datasets |
| **i2b2 / n2c2 NLP challenges** | De-identified clinical text |
| **CheXpert, MIMIC-CXR** | Chest X-rays |
| **NIH ChestX-ray14, RSNA** | Imaging |
| **Camelyon (CAMELYON16/17)** | Pathology WSI |
| **TCGA, ICGC** | Cancer genomics |
| **OMOP / OHDSI common dataset(s)** | For testing OMOP queries |

Almost every "public" health dataset has a data-use agreement. Read it.

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pydicom`, `SimpleITK`, `nibabel`, `monai` | DICOM / medical imaging |
| `pyhealth` | EHR feature engineering, models |
| `medspacy`, `negspacy`, `scispacy` | Clinical NLP |
| `lifelines`, `scikit-survival` | Survival |
| `causal-curve`, `DoWhy`, `EconML` | Causal inference |
| OHDSI tools (`ATLAS`, `HADES`) | OMOP-based cohort research |
| `synthea` | Synthetic patient generation |

## 10. References

- Friedman, C. P. & Wyatt, J. C. *Evaluation Methods in Biomedical and Health Informatics*.
- Hripcsak, G. et al. *Observational Health Data Sciences and Informatics (OHDSI): Opportunities for Observational Researchers.* Stud Health Technol Inform 216 (2015).
- Beam, A. & Kohane, I. *Big Data and Machine Learning in Health Care.* JAMA (2018).
- Goldberger, A. et al. *PhysioBank, PhysioToolkit, and PhysioNet.* Circulation 101(23) (2000).
- Obermeyer, Z. et al. *Dissecting racial bias in an algorithm used to manage the health of populations.* Science 366 (2019).
- ONC FHIR documentation — https://www.healthit.gov/topic/standards-technology/standards/fhir
- OHDSI book — https://ohdsi.github.io/TheBookOfOhdsi/
