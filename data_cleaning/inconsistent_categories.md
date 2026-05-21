# Inconsistent Categorical Values

> **TL;DR** — Free-text categorical fields acquire variants over time: `"USA"`, `"U.S.A."`, `"United States"`, `"us"`, `" USA"`. Counting them as different categories silently inflates cardinality, splits group-by aggregates, and creates "unseen-category" failures at inference. Fix early and once, with a canonical mapping that's checked into source control.

## 1. Common sources of inconsistency

- Whitespace (`"USA "` vs. `"USA"`).
- Case (`"USA"` vs. `"usa"`).
- Punctuation / abbreviations (`"U.S.A."`, `"US"`).
- Encoding artifacts (`"café"` vs. `"cafe"` vs. `"cafÃ©"`).
- Synonyms / translations (`"United States"`, `"Estados Unidos"`).
- Typos (`"Untied States"`).
- Concatenation (`"NY/NYC"`).
- Inconsistent ordering (`"Smith, John"` vs. `"John Smith"`).

## 2. Fix pipeline

```python
import pandas as pd, unicodedata, re

def normalize(s: str) -> str:
    if not isinstance(s, str):
        return s
    s = unicodedata.normalize('NFKC', s)   # unify unicode forms
    s = s.strip().lower()
    s = re.sub(r'\s+', ' ', s)             # collapse whitespace
    s = re.sub(r'[^\w\s]', '', s)          # strip punctuation
    return s

df['country_norm'] = df['country'].map(normalize)
df['country_norm'].value_counts().head(20)
```

After normalization, build a canonical mapping:

```python
COUNTRY_CANON = {
    'usa': 'United States',
    'us': 'United States',
    'united states': 'United States',
    'united states of america': 'United States',
    'uk': 'United Kingdom',
    'great britain': 'United Kingdom',
    # ...
}
df['country_clean'] = df['country_norm'].map(COUNTRY_CANON).fillna(df['country_norm'])
```

Check in the mapping. The mapping *is* the schema.

## 3. Fuzzy matching for high cardinality

When you have thousands of variants (city names, company names), build mappings semi-automatically:

```python
from rapidfuzz import process, fuzz

canonical = ['New York City', 'Los Angeles', 'San Francisco', ...]
df['city_match'], df['city_score'], _ = zip(*df['city'].map(
    lambda x: process.extractOne(x, canonical, scorer=fuzz.WRatio)
))
# Review matches with score < 90 manually
```

Tools: `rapidfuzz`, `thefuzz`, `polyfuzz`, OpenRefine.

## 4. Hierarchical categories

For hierarchies (country → state → city, category → subcategory → SKU), normalize at each level and verify consistency: every state must roll up to a valid country.

```python
assert df.groupby('state')['country'].nunique().max() == 1
```

## 5. Unseen categories at inference time

Production *will* see categories that weren't in the training set. Plan for this:

- `sklearn`'s `OneHotEncoder(handle_unknown='ignore')`.
- Target encoders need a default (overall mean, or the "Other" bucket's encoding).
- Map rare and unseen to an explicit `"OTHER"` category so the model can learn there's a tail.

## 6. Common pitfalls

1. **Hand-fixing in a notebook** without saving the mapping → next dataset re-introduces the variants.
2. **Lossy normalization** — stripping all punctuation collapses `"A-1"` and `"A1"` with `"A 1"`.
3. **Case-folding bilingual data** with locale-sensitive lowercasing (Turkish dotted/dotless `İ`/`ı`).
4. **`np.nan` vs. `""` vs. `"None"` vs. `"NA"` all coexisting** in one column.
5. **Aggregations done before normalization** silently double-count groups.

## 7. Checklist

- [ ] Listed every variant (`value_counts()` on raw).
- [ ] Normalized for whitespace, case, unicode, punctuation.
- [ ] Built a canonical mapping in source control.
- [ ] Applied the same mapping to training and inference.
- [ ] Handled hierarchy consistency.
- [ ] Decided on a policy for rare and unseen categories.
