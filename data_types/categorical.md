# Categorical Data

> **TL;DR** — Categorical data takes one of a finite set of values. The crucial distinctions are *nominal vs. ordinal* (is there a natural order?) and *low vs. high cardinality* (a handful of values, or thousands?). These determine encoding, distance, and which charts are honest.

## 1. Sub-types

### 1.1 Binary
- **Examples:** churned/retained, fraud/legitimate, click/no-click.
- **Storage:** `bool`, `int8` (0/1), `category`.
- **Special property:** mean of a 0/1 variable is the proportion — extremely useful.

### 1.2 Nominal (unordered)
- **Examples:** country, product category, blood type, hair color.
- **Cardinality:** typically low to moderate (≤ ~50).
- **Encoding for ML:** one-hot, target/mean encoding, embeddings.

### 1.3 Ordinal (ordered)
- **Examples:** education (HS < BSc < MSc < PhD), satisfaction (1–5), Likert scale, T-shirt sizes (S < M < L < XL).
- **Encoding:** integer encoding preserves order, but the *gaps* may not be equal — "very satisfied" minus "satisfied" ≠ "satisfied" minus "neutral".
- **Models:** ordinal regression handles this correctly; treating as numeric assumes equal gaps.

### 1.4 High-cardinality categorical
- **Examples:** ZIP code, product SKU, user ID, URL.
- **Problems:** one-hot encoding explodes the feature space; rare categories cause overfitting and target leakage.
- **Strategies:** target encoding (with cross-validation to avoid leakage), hashing trick, entity embeddings, grouping into "Other".

### 1.5 Hierarchical / nested
- **Examples:** product category → subcategory → SKU; country → state → city.
- **Storage:** sometimes stored as a single string ("US/CA/SF") or as separate columns.
- **Encoding:** can use level-wise encoding, hierarchical embeddings, or path strings.

### 1.6 Multi-label
- **Examples:** movie genres (a film can be both "Action" and "Comedy"), tags on a blog post.
- **Storage:** comma-separated string, JSON array, or one-hot matrix.
- **Watch out for:** never confuse with multi-class (exactly one label).

## 2. Storage representations

| Representation | Pros | Cons |
|---|---|---|
| `object` (str) | Human-readable | Memory-heavy, slow group-bys |
| `category` (pandas) | Fast, memory-efficient, optional ordering | Need to set categories explicitly |
| Integer codes | Compact | Requires lookup table to interpret |
| Hashed | Bounded memory, no lookup table | Collisions |

In pandas, `df['col'] = df['col'].astype('category')` is the right default for any column with cardinality ≤ ~50% of rows.

## 3. Common pitfalls

1. **Treating IDs as categorical features.** `user_id` has cardinality = number of users; one-hot encoding will explode and you'll memorize the training set.
2. **Encoding ordinal data as one-hot.** Throws away the order. Use integer encoding or ordinal-aware models.
3. **"Other" buckets that grow uncontrolled.** Rare categories aggregated into "Other" can become the largest single category.
4. **Inconsistent labels.** "USA", "U.S.A.", "United States", "us" are four categories until you normalize.
5. **Label drift.** A category present in training but unseen in production will crash one-hot pipelines unless you set `handle_unknown='ignore'`.
6. **Mean-encoding leakage.** Encoding a category by its target mean using the full dataset leaks the target. Always cross-validate.

## 4. Cleaning checklist

- [ ] Strip whitespace, normalize case (`"USA "` → `"usa"`).
- [ ] Map synonyms to canonical values.
- [ ] Decide what to do with unseen / rare categories.
- [ ] Confirm ordinal categories are stored with the correct order.
- [ ] Convert to `category` dtype for memory and speed.

See [`data_cleaning/inconsistent_categories.md`](../data_cleaning/inconsistent_categories.md).

## 5. Visualizations

| Question | Chart |
|---|---|
| What is the frequency of each category? | Bar chart (sorted by count) |
| What fraction of the whole does each represent? | Stacked bar; pie chart only for ≤ 5 categories |
| How does a numeric variable differ across categories? | Box plot, strip plot, violin |
| How do two categoricals relate? | Heatmap of cross-tab; mosaic plot |
| Ordinal data over time | Stacked bar, area chart |

**Avoid pie charts** when you have more than ~5 categories — humans are bad at comparing angles.

## 6. Encoding strategies for ML

| Strategy | When to use | Library |
|---|---|---|
| One-hot | Low cardinality (≤ ~20), tree models tolerant | `pd.get_dummies`, `OneHotEncoder` |
| Ordinal (integer) | True ordinal data, or trees | `OrdinalEncoder` |
| Target / mean encoding | High cardinality | `category_encoders.TargetEncoder` |
| Hashing | Very high cardinality, online learning | `FeatureHasher` |
| Entity embeddings | Deep learning, high cardinality | `nn.Embedding` |
| Frequency / count | Tree models with high cardinality | manual |
| Leave-one-out | Like target encoding but more leak-resistant | `category_encoders.LeaveOneOutEncoder` |
