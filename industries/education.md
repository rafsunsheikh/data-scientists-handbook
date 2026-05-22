# Education

> **TL;RT** — Education data spans student records (demographics, enrollment, attendance), learning interactions (LMS clickstreams, assessment outcomes), and institutional data (course catalogs, faculty, funding), all aligned to academic calendars and organizational hierarchies (student → course → section → term → institution). The distinguishing challenges are identity resolution across systems (LMS user ID vs. SIS student ID vs. state student ID), academic-year alignment (terms, semesters, quarters), course-level vs. assignment-level granularity, and the ethical constraints on using student data for predictive modeling (FERPA, GDPR, COPPA). Early warning systems and learning analytics are the highest-impact applications, but they must be designed with fairness, transparency, and student welfare in mind.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Student demographics** | Age, gender, race/ethnicity, language, disability status, free/reduced lunch |
| **Enrollment / registration** | Course enrollment, section assignment, major, year level, transfer status |
| **Attendance** | Daily attendance, tardy, excused/absent/unexcused, LMS login frequency |
| **Assessment outcomes** | Homework, quizzes, exams, projects, standardized tests |
| **LMS clickstream** | Page views, resource clicks, video plays, discussion posts, submission timestamps |
| **Gradebook** | Assignment scores, weighted categories, final grades |
| **Behavioral events** | Disciplinary actions, suspensions, honor roll, extracurricular |
| **Outcomes** | Course completion, GPA, graduation, retention, employment, further enrollment |
| **Course catalog** | Course descriptions, prerequisites, credits, department |
| **Institutional** | Faculty, budget, facilities, accreditation data |

## 2. Common sources

| System | What |
|---|---|
| **SIS (Student Information System)** | Demographics, enrollment, attendance, grades (PowerSchool, Skyward, Infinite Campus, Blackboard Student) |
| **LMS (Learning Management System)** | Canvas, Moodle, Blackboard, D2L Brightspace — course content, assignments, discussions |
| **Standardized testing** | College Board (SAT/ACT), Pearson, ETS (TOEFL, GRE), state assessments |
| **State Longitudinal Data System (SLDS)** | State-level student data across years and districts |
| **Proctoring / plagiarism** | Turnitin, Respondus, ProctorU, Honorlock |
| **Library / learning analytics** | Library circulation, database usage, LTI tool analytics |
| **IPEDS** | Integrated Postsecondary Education Data System (institutional reporting) |
| **National databases** | NSC (National Student Clearinghouse), NPSAS, EDW (US Dept of Education) |

## 3. Standard schemas and formats

### Student (typical SIS)
```
student_id, sis_user_id, state_student_id,
date_of_birth, gender, race_ethnicity,
english_learner_flag, disability_flag,
free_reduced_lunch_flag, homelessness_flag, foster_care_flag,
enrollment_status (active, withdrawn, graduated, transferred),
grade_level, school_id, district_id,
enrollment_date, withdrawal_date, withdrawal_reason
```

### LMS event (typical)
```
event_id, timestamp,
user_id (LMS), user_role (student, instructor, TA),
context_course_id, context_content_id,
event_type (content_view, assignment_submit, discussion_post, discussion_reply,
            video_play, video_pause, video_seek, resource_download, quiz_start, quiz_submit),
resource_type (page, assignment, discussion, video, quiz, file, url),
metadata (JSON: assignment_id, page_title, video_duration_sec, ...)
```

### Assessment outcome
```
student_id, course_id, section_id, term,
assignment_id, assignment_name, assignment_type (homework, quiz, exam, project),
points_earned, points_possible, weight,
submission_timestamp, graded_timestamp,
plagiarism_score, proctoring_flag
```

### IPEDS (institutional reporting)
```
institution_id, institution_name, state, control (public, private_nonprofit, private_forprofit),
enrollment_headcount, enrollment_FTE,
graduation_rate_4yr, graduation_rate_6yr,
retention_rate_freshman, admission_rate,
median_sat_score, median_act_score,
tuition_in_state, tuition_out_of_state,
financial_aid_awarded, default_rate
```

## 4. Cleaning particulars

### 4.1 Identity resolution

A student has multiple identifiers across systems:

| System | Identifier | Scope |
|---|---|---|
| **SIS** | `student_id` (e.g., 01234567) | District / institution |
| **LMS** | `sis_user_id` or `lms_user_id` | LMS instance |
| **State** | `state_student_id` | State-wide |
| **NSC** | `NSC_id` | National (across institutions) |
| **Email** | `student@domain.edu` | Communication |

**Resolution challenges:**

- **Transfers:** Student attends School A (term 1–4) then School B (term 5–8). SIS IDs differ.
- **Name changes:** Legal name change → different email, different SIS record.
- **Duplicate records:** Student enrolled twice (e.g., dual enrollment).
- **Solution:** Maintain a **student enterprise ID** table mapping all identifiers. Use probabilistic matching (name + DOB + email) when deterministic match is unavailable.

### 4.2 Academic-year alignment

Academic terms don't align with calendar years:

- **Semester system:** Fall (Aug–Dec), Spring (Jan–May).
- **Quarter system:** Fall (Sep–Dec), Winter (Jan–Mar), Spring (Mar–Jun).
- **Trimester / 4×4 block:** Various configurations.
- **Summer sessions:** Optional, variable length.

**Term coding:**

```
term_code: "202408"  # YYYYMM of term start
term_name: "Fall 2024"
term_start: "2024-08-19"
term_end: "2024-12-13"
census_date: "2024-09-13"  # Enrollment snapshot date
grade_deadline: "2024-12-10"
```

**Cleaning:**

- Align all events to the academic term, not the calendar month.
- Handle late enrollment / mid-term drops.
- Census date is the official enrollment count — use for institutional reporting.

### 4.3 Course-level vs. assignment-level granularity

| Level | Description | Use case |
|---|---|---|
| **Course** | High-level (MATH 101) | Institutional reporting, curriculum analysis |
| **Section** | Specific instance (MATH 101-01, Fall 2024) | Grade reporting, class-level analytics |
| **Assignment** | Individual task (Homework 3, Midterm Exam) | Learning analytics, early warning |
| **Question / item** | Individual test question | Item response theory, assessment design |

**Cleaning:**

- Map course codes to a standard catalog (course codes change over time).
- Prerequisites form a DAG — check for violations (student enrolled before completing prerequisite).
- Section-level vs. course-level aggregation matters for small classes.

### 4.4 Missing data patterns

| Pattern | Cause | Treatment |
|---|---|---|
| **Missing grade** | Student didn't submit | Not missing at random — behavioral signal |
| **Missing attendance** | Attendance not recorded | System issue vs. intentional |
| **Missing demographics** | Opt-out, data entry error | Structural missingness |
| **Missing LMS events** | Student didn't log in | Behavioral signal (disengagement) |
| **Missing outcomes** | Student dropped out | Censoring — use survival analysis |

### 4.5 FERPA compliance

**Family Educational Rights and Privacy Act (FERPA)** governs student data in the US:

- **Directory information:** Name, enrollment status, degree (can be disclosed).
- **Protected PII:** Student ID, grades, GPA, social security number (cannot be disclosed without consent).
- **Education records:** Any records directly related to a student.
- **Right to inspect:** Students can review their records.
- **Right to amend:** Students can request correction of inaccurate records.
- **Third-party sharing:** Requires written consent or falls under exceptions (auditors, researchers).

**Data science implications:**

- Anonymize data before sharing with researchers.
- Use data use agreements (DUAs) for external analysis.
- Avoid sharing student IDs in dashboards or reports.
- Aggregate to group level (minimum n = 5 or 10) to prevent re-identification.

### 4.6 Equity and fairness

- **Disaggregated reporting:** FERPA allows aggregate reporting by race, gender, disability — required for equity monitoring.
- **Minimum cell sizes:** Don't report metrics for groups smaller than n = 5 (FERPA).
- **Algorithmic fairness:** Early warning systems must be audited for disparate impact.
- **Bias in training data:** Historical data reflects historical inequities — models may perpetuate them.

## 5. Standard analyses

### 5.1 Early warning systems

| Analysis | Methods |
|---|---|
| **At-risk identification** | GBM, logistic regression, random survival forests |
| **Risk factors** | SHAP values, decision trees, partial dependence |
| **Intervention targeting** | Uplift modeling (who responds to intervention?) |
| **Multi-task learning** | Predict multiple outcomes (attendance, grade, behavior) jointly |

**Common risk indicators (the "ABCs"):**

- **A**ttendance (absenteeism)
- **B**ehavior (discipline, engagement)
- **C**ourse performance (F grades, low quiz scores)

### 5.2 Learning analytics

| Analysis | Methods |
|---|---|
| **Knowledge tracing** | BKT (Bayesian Knowledge Tracing), DKT (Deep Knowledge Tracing), AKT |
| **Student clustering** | K-means, GMM on engagement features |
| **Discussion network analysis** | Social network analysis (who responds to whom) |
| **Video engagement** | Play/pause/seek patterns → comprehension prediction |
| **Predictive grading** | Early prediction of final grade from midterm performance |

### 5.3 Institutional analytics

| Analysis | Methods |
|---|---|
| **Retention / completion** | Survival analysis, logistic regression |
| **Enrollment forecasting** | Time series, regression with demographic trends |
| **Curriculum effectiveness** | Value-added models, regression discontinuity |
| **Equity gaps** | Disaggregated metrics by demographic group |
| **Resource allocation** | Optimization, cost-benefit analysis |

### 5.4 Assessment analysis

| Analysis | Methods |
|---|---|
| **Item analysis** | Difficulty (p-value), discrimination (point-biserial), DIF (differential item functioning) |
| **Classical test theory** | Cronbach's alpha, reliability, validity |
| **Item response theory** | 1PL (Rasch), 2PL, 3PL models |
| **Rubric scoring** | Inter-rater reliability (Cohen's kappa, ICC) |
| **Standardized test equating** | Linking scores across test forms/years |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Enrollment to graduation | **Funnel chart** with conversion rates |
| Cohort retention | **Heatmap** (cohort × year) |
| Grade distribution | **Histogram** or **box plot** by course |
| At-risk students | **Dashboard** with risk score + key indicators |
| Engagement over time | **Line chart** (weekly LMS activity) |
| Equity gaps | **Bar chart** (metric by demographic group) |
| Course performance | **Dot plot** (mean grade ± CI by course) |
| Item difficulty | **Item characteristic curve** (IRT) |
| Student journey | **Sankey** (major → course → outcome) |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **FERPA (US)** | Student education records privacy |
| **COPPA (US)** | Under-13 data (K-12); age-gating required |
| **GDPR (EU)** | Student data, right to erasure, lawful basis |
| **IDEA (US)** | Special education data, IEP records |
| **Title IX (US)** | Gender equity, reporting requirements |
| **IPA / state privacy laws** | Student data privacy (CO, CA, UT, TX) |
| **Accreditation standards** | IPEDS reporting, outcome data |
| **Open educational resources** | OER licensing (CC-BY, CC-BY-SA) |

### Ethics

- **Transparency:** Students and parents should understand what data is collected and how it's used.
- **Consent:** Parental consent for minors; student consent for postsecondary.
- **Data minimization:** Collect only what is necessary for educational purposes.
- **Algorithmic transparency:** Early warning systems should be explainable to educators and families.
- **Beneficence:** Analytics should improve student outcomes, not just label students as "at-risk."

## 8. Public datasets

| Dataset | What |
|---|---|
| **IPEDS** | US institutional data (all postsecondary) |
| **OULAD (Open University Learning Analytics)** | 20k students, LMS data (Kaggle) |
| **NSC (National Student Clearinghouse)** | Enrollment tracking across institutions |
| **Ed-Fi Alliance** | Open education data standard + sample datasets |
| **Kaggle student datasets** | Various (student performance, engagement) |
| **PISA** | International student assessment (math, reading, science) |
| **NAEP** | National Assessment of Educational Progress ("Nation's Report Card") |
| **Common Core Data (CCD)** | US public elementary and secondary education |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Data manipulation |
| `scikit-learn`, `xgboost`, `lightgbm` | Early warning, classification |
| `lifelines`, `scikit-survival` | Survival analysis (retention, graduation) |
| `statsmodels` | IRT, regression, reliability analysis |
| `pybkt` | Bayesian Knowledge Tracing |
| `dkt` | Deep Knowledge Tracing |
| `networkx` | Discussion network analysis |
| `geopandas` | Geospatial (school catchment areas) |
| `matplotlib`, `seaborn` | Publication-quality charts |
| `streamlit`, `plotly Dash` | Equity dashboards |
| `dbt` | Data warehouse modeling |
| `Great Expectations` | Data quality validation |

## 10. References

- Baker, R. & Inventado, P. *Educational Data Mining and Learning Analytics.* In Springer Handbook of Smart Learning Systems (2014).
- Pardos, Z. & Healey, C. *Open Education Data.* KDD (2015).
- Corbett, A. & Anderson, J. *Knowledge Tracing: Modeling the Acquisition of Procedural Knowledge.* UAI (1995). (BKT)
- Piech, C. et al. *Deep Knowledge Tracing.* NeurIPS (2015). (DKT)
- FERPA Documentation — https://www2.ed.gov/policy/gen/guid/fpco/ferpa/index.html
- IPEDS Documentation — https://nces.ed.gov/ipeds/
