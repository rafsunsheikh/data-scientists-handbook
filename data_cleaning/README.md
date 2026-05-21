# Data Cleaning

> **TL;DR** — "Data cleaning" is not a single step — it's a sequence of decisions, each of which depends on the *type of data* and the *downstream use*. This section organizes those decisions both by **issue** (missing values, outliers, duplicates) and by **modality** (text, time-series, image, audio, geospatial). The companion notebooks under [`../notebooks/cleaning/`](../notebooks/cleaning/) show each step on real data.

## Why this matters

Surveys of data-science work consistently put cleaning at 60–80% of project time [^cep] [^anaconda]. More importantly, *silent* cleaning bugs — wrong-type casts, missed sentinel values, dropped duplicates that were meant to be kept — are the leading cause of irreproducible results in industry analytics.

## The cleaning pipeline (general shape)

```
1. Inspect          → shape, dtypes, summary stats, head/tail/random sample
2. Schema check     → expected columns, types, ranges, constraints
3. Parse & cast     → dates, numbers-as-strings, booleans, units
4. Deduplicate      → exact then fuzzy
5. Handle missing   → understand mechanism, then decide
6. Handle outliers  → distinguish from errors
7. Normalize text/cat → case, whitespace, synonyms
8. Validate         → run checks again; assert business rules
9. Document         → record every decision and its rationale
```

The order matters: parsing before deduplication (so "01/02/2020" and "2020-01-02" dedupe correctly), deduplication before missing-value imputation (so duplicates don't bias your fill values).

## Index — by issue

| Chapter | What it covers |
|---|---|
| [missing_values.md](missing_values.md) | MCAR/MAR/MNAR; deletion vs. imputation; per-type strategies |
| [outliers.md](outliers.md) | Detection (z, IQR, isolation forest); when to drop vs. winsorize vs. keep |
| [duplicates.md](duplicates.md) | Exact, near-duplicate, entity resolution |
| [inconsistent_categories.md](inconsistent_categories.md) | Whitespace, case, synonyms, fuzzy matching |
| [data_types_and_parsing.md](data_types_and_parsing.md) | Dates, currencies, mixed dtypes, sentinel values |
| [feature_scaling_encoding.md](feature_scaling_encoding.md) | Standardization, normalization, encoding categorical |

## Index — by modality

| Chapter | What it covers |
|---|---|
| [text_cleaning.md](text_cleaning.md) | Encoding, normalization, tokenization-safe cleanup |
| [time_series_cleaning.md](time_series_cleaning.md) | Timezones, resampling, gap-filling |
| [image_cleaning.md](image_cleaning.md) | EXIF, channels, dedup, corrupt files |
| [audio_cleaning.md](audio_cleaning.md) | Sample rate, silence, clipping, normalization |
| [geospatial_cleaning.md](geospatial_cleaning.md) | CRS, invalid geometries, geocoding QA |

## Companion notebooks

- [`../notebooks/cleaning/01_inspection_and_profiling.ipynb`](../notebooks/cleaning/01_inspection_and_profiling.ipynb)
- [`../notebooks/cleaning/02_missing_values.ipynb`](../notebooks/cleaning/02_missing_values.ipynb)
- [`../notebooks/cleaning/03_outliers.ipynb`](../notebooks/cleaning/03_outliers.ipynb)
- [`../notebooks/cleaning/04_text_cleaning.ipynb`](../notebooks/cleaning/04_text_cleaning.ipynb)

## Rules of thumb

1. **Never edit the raw data.** Read raw, write cleaned to a new location. Keep the cleaning code under version control.
2. **Cleaning is a function, not a notebook.** Encapsulate it so the same code runs on training data and on each new batch.
3. **Document every decision.** "Dropped 1,243 rows where `age < 0`" with the date and the rationale.
4. **Validate at the boundaries.** Check schema on input and on output. Tools: `pandera`, `great_expectations`, `pydantic`.
5. **Fit on train, transform on test.** Imputers, encoders, scalers must be fit on training data only. Use `sklearn` `Pipeline` or `ColumnTransformer`.
6. **Plot before and after.** Distributions, missing-value maps, count of duplicates. Eyes catch things tests miss.

[^cep]: Crowdflower / Figure Eight, *Data Scientist Report* (2017): 80% of time on data wrangling.
[^anaconda]: Anaconda *State of Data Science* (2022): 38% on data prep + cleansing.
