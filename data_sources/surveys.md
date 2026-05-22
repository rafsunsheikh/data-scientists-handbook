# Surveys as Data Sources

> **TL;DR** — Survey data is fundamentally different from behavioral or transactional data because it measures *self-reported* information, which is subject to response bias, social-desirability bias, wording effects, and mode effects. A survey is not a random sample of reality — it's a sample of the opinions of people who chose (or were selected) to answer questions in a particular way, at a particular time, in a particular mode. Proper survey analysis requires understanding the sampling design, applying weights, and using design-aware statistics that account for clustering, stratification, and unequal selection probabilities. Ignoring the survey design turns complex probability samples into pseudo-random samples, invalidating standard errors and confidence intervals.

## 1. Survey types and designs

### 1.1 Cross-sectional surveys

One-time snapshot of a population at a single point in time.

**Characteristics:**

- Single data collection wave.
- Estimates are for the population at the time of the survey.
- Cannot track individual change over time.
- Most common survey type (e.g., Gallup polls, NPS surveys).

**Use cases:**

- Market research (brand awareness, product satisfaction).
- Public opinion polling.
- Employee engagement surveys.
- Health prevalence estimates.

### 1.2 Longitudinal / panel surveys

Same individuals surveyed repeatedly over time.

**Types:**

| Type | Description | Example |
|---|---|---|
| **Cohort study** | Same age group followed over time | British Cohort Study (1958, 1970, 1991) |
| **Panel study** | Same individuals surveyed repeatedly | Panel Study of Income Dynamics (PSID) |
| **Trend study** | Different samples from same population over time | General Social Survey (GSS) |

**Advantages:**

- Observe individual-level change.
- Establish temporal ordering (cause before effect).
- Separate cohort effects from age effects.

**Disadvantages:**

- Attrition (participants drop out over time).
- Panel conditioning (participants change behavior because they're being studied).
- Expensive and slow.

### 1.3 Repeated cross-sectional surveys

Different samples from the same population at different times.

**Characteristics:**

- Cannot track individual change.
- Can track population-level change.
- Easier and cheaper than panel surveys.
- Sample composition may change over time.

**Examples:**

- Behavioral Risk Factor Surveillance System (BRFSS) — annual US health survey.
- European Social Survey (ESS) — biennial.
- Programme for International Student Assessment (PISA) — triennial.

## 2. Sampling methods

### 2.1 Probability sampling (random)

Every member of the population has a known, non-zero probability of selection. Enables statistical inference to the population.

| Method | How it works | When to use |
|---|---|---|
| **Simple random** | Random draw from full population list | Small, well-defined population |
| **Systematic** | Every Nth person from a sorted list | When list is randomly ordered |
| **Stratified** | Divide into strata, sample within each | Ensure representation of subgroups |
| **Cluster** | Randomly select clusters (schools, villages), survey all within | Large geographic areas |
| **Multi-stage** | Cluster → sub-cluster → individual | National surveys (state → district → household → person) |
| **Probability proportional to size (PPS)** | Larger clusters have higher selection probability | Unequal cluster sizes |

**Sampling frame:** The actual list from which the sample is drawn. Must approximate the target population. Mismatches cause coverage error.

### 2.2 Non-probability sampling

Selection probabilities are unknown. Cannot compute sampling errors. Useful for exploratory work but not for population inference.

| Method | Description | Risk |
|---|---|---|
| **Convenience** | Survey whoever is available | Selection bias |
| **Snowball** | Participants recruit other participants | Hard-to-reach populations; network bias |
| **Quota** | Non-random selection to match population proportions | Same as convenience within quotas |
| **Respondent-driven** | Snowball with weighting adjustments | Complex weighting; still debated |
| **Online panel** | Pre-recruited panel (Qualtrics, Respondent, Swagbucks) | Self-selection; not representative |

### 2.3 Sample size

**Factors:**

- **Population size:** Matters less than people think. A sample of 1,500 gives ~±2.5% margin of error for a proportion regardless of whether the population is 100,000 or 100,000,000.
- **Confidence level:** 95% is standard (z = 1.96). 99% requires larger sample.
- **Margin of error:** Half the confidence interval width. Common targets: ±2%, ±3%, ±5%.
- **Expected proportion:** Worst case is p = 0.5 (maximum variance). If you expect p = 0.1, you need a smaller sample.
- **Design effect:** Complex sampling (cluster, stratified) increases variance. Design effect (DEFF) of 1.5–2.0 is common.

**Formula (simple random):**

```
n = (z² × p × (1-p)) / E²

where:
  z = z-score for confidence level (1.96 for 95%)
  p = expected proportion (0.5 for worst case)
  E = margin of error
```

Example: 95% confidence, ±3% margin of error, p = 0.5 → n = 1,067.

**Adjust for design effect:** `n_adjusted = n × DEFF` (e.g., 1,067 × 1.5 = 1,600).

**Adjust for non-response:** `n_adjusted = n / response_rate` (e.g., 1,600 / 0.3 = 5,333 invitations needed for 30% response).

## 3. Question design

### 3.1 Question wording effects

The same question worded differently produces different answers.

**Common pitfalls:**

- **Leading questions:** "Don't you agree that taxes are too high?" vs. "What do you think about tax levels?"
- **Double-barreled questions:** "Do you support increasing education and healthcare funding?" (two issues in one).
- **Ambiguous terms:** "Often," "sometimes," "fairly" — different respondents interpret differently.
- **Technical jargon:** "Do you understand your deductible?" — not everyone knows what a deductible is.
- **Negatively worded questions:** "I am not satisfied with my job" — respondents may miss the "not."

**Best practices:**

- Pre-test questions with cognitive interviews.
- Use neutral wording.
- Define technical terms.
- Keep questions short and simple.
- Pilot test with 10–20 people before full launch.

### 3.2 Response scales

| Scale type | Example | Use case |
|---|---|---|
| **Dichotomous** | Yes / No | Factual questions |
| **Likert** | Strongly disagree → Strongly agree (5 or 7 point) | Attitudes, opinions |
| **Semantic differential** | Good ← → Bad (7 point) | Brand perception, emotions |
| **Numeric rating** | 0–10 scale | NPS, satisfaction |
| **Frequency** | Never, Rarely, Sometimes, Often, Always | Behavior |
| **Rank order** | Rank these 5 brands from most to least preferred | Preference |
| **Open-ended** | Text box | Exploratory, qualitative |

**Likert scale considerations:**

- **Odd vs. even points:** Odd allows neutral midpoint; even forces a direction.
- **Label every point, not just endpoints:** Improves consistency.
- **Symmetric:** Equal positive and negative options.
- **Treatment as numeric:** Common but debated. Treating Likert as interval data (mean = 3.7) is standard practice but technically assumes equal intervals between points.

### 3.3 Question ordering effects

- **Primacy effect:** Earlier questions influence later answers.
- **Context effects:** Previous questions prime certain thoughts.
- **Screening questions:** "Do you use social media?" → if yes, "Which platforms?" — skip logic matters.
- **Reverse-scored items:** In multi-question scales, reverse some items to reduce acquiescence bias.

### 3.4 Open-ended vs. closed-ended

| Dimension | Closed-ended | Open-ended |
|---|---|---|
| **Analysis** | Quantitative, fast | Qualitative, slow (coding required) |
| **Response rate** | Higher | Lower |
| **Depth** | Limited to options | Unlimited |
| **Bias** | Response set bias | Respondent effort bias |
| **Use** | Quantitative analysis | Exploratory, follow-up |

## 4. Modes of administration

### 4.1 Mode effects

The survey mode affects responses, even for the same question. Cross-mode comparisons are problematic.

| Mode | Advantages | Disadvantages | Cost |
|---|---|---|---|
| **Face-to-face** | High response rate, complex questions, observation | Expensive, interviewer bias, slow | Very high |
| **Telephone** | Fast, geographic reach | Short surveys only, declining response | High |
| **Mail / paper** | No interviewer bias, visual aids | Low response rate, slow, no validation | Medium |
| **Online** | Cheap, fast, large samples, visual | Self-selection bias, digital divide | Low |
| **SMS / text** | High open rate, mobile-friendly | Very short only, no visuals | Low |
| **Interactive voice response (IVR)** | Automated phone, no interviewer | Limited to simple questions | Medium |

### 4.2 Mixed-mode surveys

Combining modes to improve coverage and response:

- **Online primary + mail follow-up** for non-respondents.
- **Phone screen + online survey** for complex instruments.
- **Trade-off:** Mode effects complicate analysis. Use mode as a covariate or calibrate responses across modes.

## 5. Response rates and non-response bias

### 5.1 Calculating response rate

**RR1 (AAPOR Standard):**

```
Response Rate = Completed Interviews / (Completed + Partial + Presumed Non-eligible + Screened-out eligibles + Refusals)
```

**Simple approximation:**

```
Response Rate = Completed Surveys / (Completed Surveys + Non-respondents)
```

**Current US response rates:**

- Telephone: 5–10% (declining since 1990s).
- Online panels: 2–5% (for unsolicited invitations).
- Mail: 15–30%.
- Face-to-face: 40–60%.

### 5.2 Non-response bias

Non-respondents differ from respondents in ways that affect the results.

**Detection methods:**

- **Early vs. late respondents:** Late respondents approximate non-respondents (Armstrong & Overton, 1977). Compare early vs. late on key variables.
- **Wave analysis:** Compare responses across invitation waves.
- **External benchmarks:** Compare sample demographics to population benchmarks (census).
- **Follow-up subsample:** Survey a random subset of non-respondents via a different mode.

**Mitigation:**

- Weighting (post-stratification, raking).
- Follow-up invitations (multiple waves).
- Incentives (cash, gift cards, entry into prize draw).
- Personalized invitations.
- Shorter surveys.

## 6. Weighting and post-stratification

### 6.1 Why weight?

A raw sample rarely matches the population. Weighting adjusts for:

- **Unequal selection probabilities** (stratified, cluster sampling).
- **Non-response** (subgroups with lower response rates).
- **Coverage error** (under-sampled populations).
- **Sample matching** (align sample to population benchmarks).

### 6.2 Post-stratification weighting

Align sample margins to known population margins.

```
weight = population_proportion / sample_proportion

Example:
  Population: 60% female, 40% male
  Sample:     50% female, 50% male
  Weight:     female = 60/50 = 1.2, male = 40/50 = 0.8
```

### 6.3 Raking (iterative proportional fitting)

Adjust weights to match multiple margins simultaneously (gender, age, region, education).

**Process:**

1. Start with base weights (inverse of selection probability).
2. Adjust to match population margin on variable 1 (e.g., gender).
3. Adjust to match population margin on variable 2 (e.g., age).
4. Repeat until convergence (usually 3–5 iterations).

**Tools:**

- R: `survey` package (`rake` function), `survey` weighting.
- Python: `samplics` weighting module.

### 6.4 Calibration weighting

Generalization of raking that minimizes weight change while matching population totals.

### 6.5 Weight trimming

Extreme weights (e.g., a subgroup with 1% response rate gets weight = 100) inflate variance. Trim weights at percentiles (e.g., 1st and 99th).

## 7. Common biases

### 7.1 Social-desirability bias

Respondents answer in a way that makes them look good.

**Affected topics:** Voting, drug use, sexual behavior, income, health behaviors (smoking, exercise).

**Mitigation:**

- Anonymous surveys.
- Self-administered (not interviewer-administered).
- Randomized response technique (answer sensitive question only if a random mechanism says so).
- Indirect questioning.

### 7.2 Acquiescence bias

Tendency to agree with statements regardless of content.

**Mitigation:**

- Reverse-scored items.
- Balanced scales (equal positive and negative options).
- Force-choice format (choose between two equally desirable options).

### 7.3 Response-set bias

Tendency to select the same point on a scale regardless of content (e.g., always choosing "3" on a 5-point scale).

**Mitigation:**

- Vary scale direction.
- Vary question content.
- Attention checks.

### 7.4 Satisficing

Respondents give minimal effort — pick the first plausible answer, shorten open-ends, straight-line on grids.

**Mitigation:**

- Short surveys.
- Engaging design (progress bars, varied question types).
- Attention checks (e.g., "Select 'Strongly disagree' to show you're reading").
- Instructed responses ("Please select 'Agree' for this question").

### 7.5 Recall bias

Respondents inaccurately remember past events.

**Affected by:** Time since event, importance of event, frequency of event.

**Mitigation:**

- Use recall aids (calendars, life-event anchors).
- Shorten recall period.
- Use event-history calendars.
- Validate against records when possible.

### 7.6 Selection bias

The sampling frame doesn't cover the target population.

**Examples:**

- Online panel misses people without internet.
- Phone survey misses people who only use cell phones (when using landline frame).
- Web survey misses people who don't visit the website.

## 8. Privacy and consent

### 8.1 Informed consent

- Participants must understand the purpose, risks, and benefits.
- Consent must be voluntary.
- Participants can withdraw at any time.
- Required for IRB-reviewed research.

### 8.2 Regulations

| Regulation | Scope |
|---|---|
| **IRB (US)** | Institutional Review Board approval for research with human subjects |
| **GDPR (EU)** | Personal data, right to erasure, lawful basis for processing |
| **HIPAA (US)** | Health information (if survey collects health data) |
| **COPPA (US)** | Under-13 data (age gating required) |
| **Belmont Report** | Ethical principles: respect, beneficence, justice |

### 8.3 Data anonymization

- Remove direct identifiers (name, email, address).
- Consider k-anonymity (each record indistinguishable from k−1 others).
- Be cautious with combinations of demographic variables (can re-identify).

## 9. Public survey datasets

| Dataset | Country | Frequency | Topics |
|---|---|---|---|
| **General Social Survey (GSS)** | US | Annual | Attitudes, demographics |
| **American Community Survey (ACS)** | US | Annual | Demographics, housing, economy |
| **BRFSS** | US | Annual | Health behaviors, chronic conditions |
| **NHANES** | US | Annual | Health, nutrition (with physical exams) |
| **Panel Study of Income Dynamics (PSID)** | US | Annual (since 1968) | Income, wealth, family |
| **European Social Survey (ESS)** | Europe | Biennial | Attitudes, demographics |
| **World Values Survey (WVS)** | Global | Wave every ~5 years | Values, beliefs |
| **Eurostat** | EU | Monthly/annual | Economic, social statistics |
| **OECD Data** | OECD countries | Annual | Education, health, economy |
| **PISA** | 80 countries | Triennial | Student achievement |
| **Demographic and Health Surveys (DHS)** | Developing countries | Irregular | Fertility, health, family planning |
| **IPUMS** | Multiple (historical) | Varies | Census microdata harmonized |

## 10. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `samplics` | Survey-weighted statistics in Python |
| `statsmodels` | Regression with survey weights |
| `pandas` | Data manipulation |
| `numpy` | Weighted computations |
| `survey` (R) | Comprehensive survey analysis (R package) |
| `weights` (R) | Weighting and calibration (R package) |
| `QualtricsPY` | Qualtrics API client |
| `pySurvey` | Survey data handling |
| `Jupyter` + `matplotlib` / `seaborn` | Analysis and visualization |

### Weighted statistics in Python

```python
import samplics
from samplics.survey_designs import HorvitzThompson
from samplics.population_totals import PopulationTotals

# Define population totals for raking
pop_totals = {"gender": {"M": 0.5, "F": 0.5}, "age_group": {"18-29": 0.25, "30-44": 0.35, "45+": 0.40}}

# Create population totals object
pop = PopulationTotals(pop_totals, domains=None)

# Get survey design with raking
survey_design = HorvitzThompson(sample_df, sample_weights=None,
                                 population_totals=pop,
                                 raking=True)

# Estimate proportions with correct standard errors
estimates = survey_design.get_total("satisfaction")
print(estimates.summary())
```

## 11. References

- Dillman, D. A., Smyth, J. D., & Christian, L. M. *Internet, Phone, Mail, and Mixed-Mode Surveys: The Tailored Design Method*.
- Groves, R. M. et al. *Survey Methodology*. Wiley (2009).
- Lohr, S. L. *Sampling: Design and Analysis*. Wiley.
- Armstrong, J. S. & Overton, R. S. *Estimating Nonresponse Bias in Mail Surveys*. Journal of Marketing Research (1977).
- IPA (Insights Association) *Best Practices on Responding to Surveys and Reporting Usage*.
- AAPOR (American Association for Public Opinion Research) *Standard Definitions: Final Dispositions of Case Codes*.
- Wolter, K. M. *Introduction to Variance Estimation*. Springer.
