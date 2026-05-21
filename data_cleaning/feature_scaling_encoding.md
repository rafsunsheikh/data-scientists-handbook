# Feature Scaling and Encoding

> **TL;DR** — Scaling and encoding are the bridge between cleaned data and a model. The choices are simple if you know two things: which model class you'll use, and whether the column is numeric, ordinal, or nominal. Wrap everything in a `Pipeline` so you can't accidentally fit on the test set.

## 1. Does scaling matter for your model?

| Model class | Needs scaling? |
|---|---|
| Linear regression, logistic regression | Yes (for regularization / interpretability of coefficients) |
| SVM, KNN | Yes (distance-based) |
| Neural networks | Yes (gradient stability) |
| PCA, k-means, t-SNE, UMAP | Yes (distance / variance based) |
| Naive Bayes | No |
| Decision trees, random forests, gradient boosting | No (split on each feature independently) |
| Most ranking models | Usually no |

If you use trees, you can skip scaling entirely.

## 2. Numerical scaling options

| Scaler | Formula | When |
|---|---|---|
| `StandardScaler` | `(x − μ) / σ` | Roughly Gaussian, no extreme outliers |
| `MinMaxScaler` | `(x − min) / (max − min)` | Bounded inputs (NN with sigmoid output) |
| `MaxAbsScaler` | `x / max(|x|)` | Sparse data; preserves zeros |
| `RobustScaler` | `(x − median) / IQR` | Outlier-heavy data |
| `QuantileTransformer` | Maps to uniform or Gaussian via quantiles | Non-monotonic distribution issues |
| `PowerTransformer` (Yeo-Johnson / Box-Cox) | Make Gaussian-ish | Skewed data, linear models |
| `Normalizer` | Row-wise norm to 1 | Text TF-IDF, cosine similarity |

```python
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

pipe = Pipeline([
    ('scale', StandardScaler()),
    ('model', LogisticRegression()),
])
pipe.fit(X_train, y_train)   # fits scaler on train only
```

## 3. Log / Box-Cox for skewed features

Right-skewed columns (income, file size, latency) become much more model-friendly under `log1p`:

```python
import numpy as np
X['income_log'] = np.log1p(X['income'])
```

For columns with both positive and negative values, Yeo-Johnson is the generalization (`PowerTransformer(method='yeo-johnson')`).

## 4. Categorical encoding

(See [`../data_types/categorical.md`](../data_types/categorical.md) for context on the data type.)

| Encoding | When | Library |
|---|---|---|
| One-hot | Low cardinality (≤ ~20), nominal | `pd.get_dummies`, `OneHotEncoder` |
| Ordinal (integer) | True ordinal, or tree models | `OrdinalEncoder` |
| Target / mean | High cardinality, smoothed | `category_encoders.TargetEncoder` (cross-validated) |
| Frequency / count | Trees, high cardinality | manual or `category_encoders.CountEncoder` |
| Hashing | Very high cardinality, online learning | `FeatureHasher` |
| Embeddings | DL, high cardinality | `nn.Embedding` |
| Leave-one-out | Less leakage than target encoding | `category_encoders.LeaveOneOutEncoder` |
| WOE (weight of evidence) | Credit scoring, binary classification | `category_encoders.WOEEncoder` |

## 5. Putting it together with `ColumnTransformer`

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

num_cols = ['age', 'income', 'tenure']
cat_cols = ['country', 'segment']

pre = ColumnTransformer([
    ('num', StandardScaler(), num_cols),
    ('cat', OneHotEncoder(handle_unknown='ignore'), cat_cols),
])

pipe = Pipeline([('pre', pre), ('clf', LogisticRegression(max_iter=1000))])
pipe.fit(X_train, y_train)
```

This pattern is the safest default — you can never accidentally fit on the test set.

## 6. Common pitfalls

1. **Fitting the scaler on the full dataset before splitting** — leaks test statistics into training.
2. **Forgetting to apply the same encoding at inference** — production breaks on unseen categories or unscaled inputs.
3. **One-hot encoding high-cardinality columns** — explodes the feature space, blows out memory.
4. **Ordinal-encoding nominal data** — implies a fake order (`country=1, 2, 3, ...`) that linear models will use.
5. **Mean / target encoding without cross-validation** — leaks the target.
6. **Scaling tree models** — harmless but wasted compute and obscures interpretability.
7. **Standardizing dummies (one-hot columns)** — usually unnecessary, sometimes harmful for sparsity.

## 7. Checklist

- [ ] Decided whether the chosen model needs scaling.
- [ ] Chose a scaler matching the data distribution and outlier profile.
- [ ] Log-transformed right-skewed columns where helpful.
- [ ] Chose categorical encodings matching cardinality and model class.
- [ ] Wrapped everything in `Pipeline` / `ColumnTransformer`.
- [ ] Confirmed `fit` only sees training data.
- [ ] Confirmed inference applies the same pipeline.
