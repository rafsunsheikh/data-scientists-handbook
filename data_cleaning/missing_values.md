# Missing Values

> **TL;DR** — Before choosing a method, classify *why* the value is missing (MCAR, MAR, MNAR). Then pick the simplest method that respects that mechanism and doesn't leak across train/test. Mean-imputation is almost never the right answer for anything other than a quick baseline.

## 1. Missingness mechanisms (Rubin, 1976 [^rubin])

| Mechanism | Meaning | Example | Safe to ignore? |
|---|---|---|---|
| **MCAR** — Missing Completely At Random | Missingness unrelated to any value (observed or not) | sensor randomly drops packets | Yes — listwise deletion unbiased but inefficient |
| **MAR** — Missing At Random | Missingness depends on *observed* data | older users skip the "income" question | Imputation conditioned on observed data is unbiased |
| **MNAR** — Missing Not At Random | Missingness depends on the *unobserved* value itself | high earners refuse to report income | Hard — needs explicit modeling of the missingness process |

You can never *prove* a mechanism from data alone — you need domain knowledge. But you can falsify MCAR with a simple test: is missingness correlated with other observed variables?

## 2. Detection (Python)

```python
import pandas as pd
import missingno as msno

df.isna().sum().sort_values(ascending=False)        # count per column
df.isna().mean().sort_values(ascending=False)       # fraction per column
df.isna().sum().sum()                               # total
df.isna().any(axis=1).sum()                         # rows with any missing

msno.matrix(df)        # visualize pattern by row
msno.heatmap(df)       # correlations of missingness across columns
msno.dendrogram(df)    # cluster columns by missingness pattern
```

A common red flag: **structured missingness** — entire blocks of rows have a column missing because they came from a different source or system. This is rarely MCAR.

## 3. Strategies by data type

### 3.1 Numerical

| Strategy | When |
|---|---|
| Drop rows (listwise) | <5% missing, MCAR |
| Drop column | >50% missing, low importance |
| Mean | Quick baseline only; distorts variance |
| Median | Skewed data |
| Mode | Discrete numerical |
| KNN imputer | Few correlated columns, moderate size |
| MICE / IterativeImputer | Multiple columns missing, MAR |
| Model-based (regression, GBM) | When you have signal in other columns |
| Forward / back fill | Time series only |
| Indicator + impute | Always pair imputation with a "was_missing" flag |

### 3.2 Categorical

| Strategy | When |
|---|---|
| Mode | Quick baseline |
| New category `"Missing"` | Most robust default; lets models learn the pattern |
| Predicted from other features | High signal in covariates |

### 3.3 Time series

| Strategy | When |
|---|---|
| Forward fill | Slow-changing values (e.g., daily price) |
| Linear interpolation | Smooth signals |
| Spline interpolation | Smooth signals with curvature |
| Seasonal decomposition + interpolate residual | Strong seasonality |
| Kalman smoother / state space | Principled, handles uncertainty |
| Domain-specific (e.g., 0 for stockout) | When zero is the truth |

### 3.4 Text
- Empty string `""` is usually the right "missing".
- Beware of strings like `"NULL"`, `"None"`, `"N/A"`, `"-"`, `"nan"` that arrived as text and aren't recognized as missing.

### 3.5 Image / audio
- Missing file → drop record or substitute a placeholder with explicit `missing=True` flag.
- Partial corruption: try-decode and reject.

## 4. The leak you'll create if you're not careful

```python
# WRONG — fits imputer on full data, leaks test info into train
df['x'] = SimpleImputer().fit_transform(df[['x']])
train, test = train_test_split(df)

# RIGHT — fit on train only
train, test = train_test_split(df)
imputer = SimpleImputer().fit(train[['x']])
train['x'] = imputer.transform(train[['x']])
test['x']  = imputer.transform(test[['x']])

# BEST — wrap in a Pipeline so it can't be done wrong
pipe = Pipeline([('impute', SimpleImputer()), ('model', LogisticRegression())])
pipe.fit(X_train, y_train)
```

## 5. The indicator trick

Whenever you impute, add a binary `was_missing_<col>` column. Tree-based models in particular often find that missingness itself is predictive. This is essential under MNAR.

```python
for col in cols_with_missing:
    df[f'{col}_was_missing'] = df[col].isna().astype(int)
df = df.fillna(df.median(numeric_only=True))
```

## 6. Common pitfalls

1. **Treating sentinel values as observed.** `-1`, `999`, `9999-12-31`, `0` (for "unknown") will bias every statistic.
2. **Imputing the target.** Don't. Drop those rows or treat as a semi-supervised problem.
3. **Forgetting to apply the same imputation at inference.** Production breaks the day the first NaN arrives.
4. **Imputing categorical with mean.** Doesn't even make sense, and `sklearn` will silently coerce to float.
5. **Imputing time series with global mean** instead of using temporal structure.
6. **Reporting model accuracy on imputed data** without noting it.

## 7. Companion notebook

[`../notebooks/cleaning/02_missing_values.ipynb`](../notebooks/cleaning/02_missing_values.ipynb)

[^rubin]: Rubin, D. B. (1976). *Inference and missing data.* Biometrika, 63(3), 581–592.
