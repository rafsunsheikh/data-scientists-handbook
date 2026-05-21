# Duplicates

> **TL;DR** — "Duplicate" has three meanings: exact, near, and same-entity. Each requires a different tool. Always deduplicate *before* you train/test split (to avoid leakage) and *before* you compute aggregates (to avoid double-counting).

## 1. Three kinds of duplicates

### 1.1 Exact duplicates
All columns identical. Caused by ingestion bugs, retries, joining the same source twice.

```python
df.duplicated().sum()
df = df.drop_duplicates()
```

Consider only the columns that *should* be unique:

```python
df = df.drop_duplicates(subset=['user_id', 'event_time'])
```

### 1.2 Near-duplicates
Slightly different — whitespace, case, punctuation, typos.

For text:
- Normalize (lowercase, strip whitespace, remove punctuation) **then** dedupe.
- Hashing: MD5 / SHA1 of the normalized string.
- Locality-sensitive hashing (MinHash + LSH) for large corpora.

For images:
- Perceptual hash (`imagehash.phash`), then group by Hamming distance.

For audio:
- Audio fingerprinting (Chromaprint / `acoustid`).

### 1.3 Entity resolution (a.k.a. record linkage, deduplication)
Same real-world entity recorded differently across sources or time. Most expensive class.

Example: `"John Smith, 123 Main St, NYC"` and `"J. Smith, 123 Main Street, New York"`.

Pipeline:
1. **Blocking** — group candidate matches (e.g., same ZIP code) to avoid `O(n²)` comparison.
2. **Pairwise comparison** — string similarity (Jaro-Winkler, Levenshtein), numerical proximity, semantic embeddings.
3. **Decision** — rule-based threshold, or a classifier trained on labeled pairs (Fellegi-Sunter, Splink, dedupe.io).
4. **Clustering** — transitive closure of matches.
5. **Resolution** — pick or merge attributes (survivorship rules).

Tools: `recordlinkage`, `dedupe`, `splink`, `zingg`.

## 2. Common pitfalls

1. **Deduping after splitting train/test** → same record can land in both, inflating metrics.
2. **Deduping with the wrong subset** → e.g., dropping rows that have the same `user_id` but represent different sessions.
3. **Survivorship bugs.** When merging duplicates, which row's value wins? Most-recent? Most-complete? Document the rule.
4. **Case-sensitive dedupe on case-inconsistent data** → `"USA"` and `"usa"` survive as two rows.
5. **Hashing without normalization first.** `"john "` and `"John"` hash differently.
6. **Time-window confusion.** "Same event twice within 1 second" is a duplicate; "twice in a day" usually isn't.

## 3. Order of operations

```
raw data
  → parse types
  → normalize text (case, whitespace, unicode)
  → exact dedupe on a sensible subset
  → fuzzy / entity-resolution dedupe
  → then split, aggregate, model
```

## 4. Checklist

- [ ] What is the *intended* uniqueness key? (Document it.)
- [ ] How many exact duplicates exist on that key?
- [ ] Are there near-duplicates that normalization would catch?
- [ ] Do I need entity resolution across sources?
- [ ] Have I deduplicated before train/test split?
- [ ] If merging, what is the survivorship rule, and is it audit-logged?
