# Outliers

> **TL;DR** — Outliers can be (a) data errors, (b) genuine rare events, or (c) the most interesting observations in the dataset. Detection is the easy part. The right *response* depends entirely on the cause and the downstream use. Defaulting to "drop anything beyond 3 sigma" loses real signal and is rarely the right choice.

## 1. What an outlier *is*

There is no universal definition. Useful working definitions:

- **Statistical** — far from the mean / median by some criterion.
- **Distributional** — low probability under an assumed distribution.
- **Density-based** — in a sparse region of feature space.
- **Contextual** — outlier relative to a subgroup (e.g., 200 km/h on a city street, but normal on the autobahn).
- **Collective** — a subsequence that's anomalous together, even if individual points aren't (time series).

## 2. Detection — univariate

| Method | How | Use when |
|---|---|---|
| **z-score** | `\|(x - μ) / σ\| > 3` | Roughly normal, no extreme outliers (μ, σ are themselves sensitive) |
| **Modified z-score** | Uses median and MAD instead of mean and SD | More robust default |
| **IQR rule (Tukey)** | `x < Q1 − 1.5·IQR` or `x > Q3 + 1.5·IQR` | Skewed data; standard box-plot definition |
| **Percentile** | Top/bottom 1% or 0.5% | Rough cap |
| **ECDF inspection** | Visual | Always do this as a sanity check |

```python
import numpy as np
def modified_z_score(x):
    median = np.median(x)
    mad = np.median(np.abs(x - median))
    return 0.6745 * (x - median) / mad   # |score| > 3.5 ≈ outlier
```

## 3. Detection — multivariate

| Method | Notes |
|---|---|
| **Mahalanobis distance** | Accounts for covariance; assumes elliptical distribution |
| **Isolation Forest** | Tree-based, no distribution assumptions, scales well |
| **Local Outlier Factor (LOF)** | Density-based, finds locally anomalous points |
| **DBSCAN** | Points labeled as noise = outliers |
| **One-Class SVM** | Boundary of "normal" region |
| **Autoencoder reconstruction error** | High-dim, unstructured (images, embeddings) |
| **Robust covariance (EllipticEnvelope)** | Elliptical, robust to contamination |

```python
from sklearn.ensemble import IsolationForest
iso = IsolationForest(contamination=0.01, random_state=0).fit(X)
df['is_outlier'] = iso.predict(X) == -1
```

## 4. Detection — time series

- **Rolling z-score** with a robust estimator (median + MAD).
- **STL decomposition residuals** beyond a threshold.
- **Prophet / state-space residuals** beyond a confidence band.
- **Matrix profile** (`stumpy`) for unusual subsequences.
- **Change-point detection** (`ruptures`, CUSUM) for level shifts.

## 5. Response — what to do with detected outliers

| Cause | Response |
|---|---|
| Data-entry error (typo, unit mistake) | Fix if possible; otherwise drop with note |
| Sensor failure | Drop or impute with a `was_outlier` flag |
| Legitimate rare event | **Keep** — it's often the signal you care about |
| Heavy-tailed distribution | Transform (log, Box-Cox); don't drop |
| Model that can't handle them | Winsorize or use a robust model instead of trimming the data |

Decision rule: **never drop an outlier silently**. Either keep it, fix it (with audit log), or flag it for review.

## 6. Robust alternatives to dropping

| Need | Robust method |
|---|---|
| Summary statistic | Median, MAD, trimmed mean |
| Regression | Huber regression, Theil-Sen, RANSAC |
| Distance | Mahalanobis, Manhattan |
| Scaling | `RobustScaler` (median / IQR) |
| Loss function | Huber loss, log-cosh |

## 7. Common pitfalls

1. **3-sigma rule on skewed data.** A log-normal distribution will have many "3-sigma outliers" that are actually normal.
2. **Dropping outliers from the test set.** Production will see them; your evaluation should too.
3. **Outlier removal that uses the target.** Removing rows because their prediction is bad is data dredging.
4. **Confusing outliers with mistakes.** "$1M house in Beverly Hills" is not an error.
5. **Stacked transformations that hide outliers.** Log + winsorize + standardize and you've forgotten which step removed what.
6. **Reporting model performance after aggressive outlier removal** without disclosure.

## 8. Companion notebook

[`../notebooks/cleaning/03_outliers.ipynb`](../notebooks/cleaning/03_outliers.ipynb)

## 9. References

- Hawkins, D. M. (1980). *Identification of Outliers.*
- Aggarwal, C. C. *Outlier Analysis*, 2nd ed.
- Hyndman & Athanasopoulos, *FPP3*, §3.5 (time series).
