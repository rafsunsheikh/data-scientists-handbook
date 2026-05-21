# Data Types and Parsing

> **TL;DR** — Many "bad data" problems are really *bad parsing* problems. A column of numbers loaded as strings, a date misread as a string, a 0/1 boolean stored as text — these silently corrupt every downstream step. Fix them at load time and assert the schema.

## 1. Dtype audit

```python
df.dtypes
df.info(memory_usage='deep')
```

Red flags:

- `object` dtype on a column you expect to be numeric → strings mixed in, or scientific notation, or units appended.
- `float64` on what should be `int` → there are NaNs forcing the float. Use pandas `Int64` (nullable).
- `int64` on what should be `category` → wasted memory, slow group-bys.
- All-zero column with `float64` → likely a parsing failure or sentinel.

## 2. Numbers stored as strings

Common culprits:

- Thousands separator: `"1,234"` → use `thousands=','` or `str.replace`.
- Decimal separator: `"1.234,56"` (European) — set `decimal=','`.
- Currency symbols: `"$1,234.56"` — strip with regex.
- Units appended: `"3.4 kg"` — extract with regex.
- Percentages: `"15%"` → strip, divide by 100.
- Trailing whitespace: `" 42 "` — strip first.
- Scientific notation: `"1e-05"` — usually handled automatically.

```python
df['price'] = (df['price']
               .str.replace(r'[\$,]', '', regex=True)
               .astype(float))
```

## 3. Dates

The single largest source of parsing bugs. Defensive defaults:

```python
df['ts'] = pd.to_datetime(df['ts'], format='ISO8601', errors='coerce', utc=True)
```

- **Specify `format`** when you know it. Inferring is slow and ambiguous (`01/02/2020` — Jan 2 or Feb 1?).
- **`errors='coerce'`** turns unparseable values into `NaT` rather than raising.
- **`utc=True`** prevents timezone surprises later.
- Audit with `df['ts'].isna().sum()` — high → format guess was wrong.

Locale issue: `01/02/2020` in US = Jan 2; in EU = Feb 1. Always know the source.

Two-digit years: `2/3/45` — 1945 or 2045? Force four-digit years upstream if you can.

## 4. Booleans

Watch for: `"true"`, `"True"`, `"TRUE"`, `"1"`, `"yes"`, `"Y"`, `"t"`, `True`, `1`. All the same logical value, different parses.

```python
BOOL_MAP = {'true': True, 't': True, 'yes': True, 'y': True, '1': True,
            'false': False, 'f': False, 'no': False, 'n': False, '0': False}
df['flag'] = df['flag'].astype(str).str.lower().map(BOOL_MAP)
```

## 5. Sentinel values

Legacy systems encode "missing" or "unknown" as:

- `-1`, `999`, `9999` (numeric)
- `9999-12-31`, `1900-01-01`, `0000-00-00` (dates)
- `""`, `"NA"`, `"N/A"`, `"NULL"`, `"None"`, `"-"`, `"?"` (text)

These will silently distort every statistic. Make `read_csv` recognize them:

```python
pd.read_csv('file.csv', na_values=['NA', 'N/A', 'NULL', '-', '?', '9999'])
```

After load, double-check:

```python
df.describe()        # numerical: look for suspicious min/max
df.select_dtypes('object').apply(lambda s: s.unique()[:10])
```

## 6. Mixed dtypes in one column

A column can end up as `object` because some rows are numeric strings and some have units. Detect:

```python
df['x'].apply(type).value_counts()
```

Then handle each subtype explicitly.

## 7. Encoding

Read CSV with the right encoding:

```python
pd.read_csv('file.csv', encoding='utf-8')        # default
pd.read_csv('file.csv', encoding='latin-1')      # legacy Windows / old data
pd.read_csv('file.csv', encoding='utf-16')       # Excel exports on Windows
```

Use `chardet` or `charset_normalizer` if unknown.

## 8. Schema validation

Assert what you expect *every* load:

```python
import pandera as pa
schema = pa.DataFrameSchema({
    'user_id': pa.Column(int),
    'event_time': pa.Column(pa.DateTime, nullable=False),
    'amount': pa.Column(float, checks=pa.Check.ge(0)),
    'country': pa.Column(str, checks=pa.Check.isin(KNOWN_COUNTRIES)),
})
df = schema.validate(df)
```

`great_expectations` is similar with more reporting infrastructure.

## 9. Checklist

- [ ] Verified dtype of every column.
- [ ] Replaced sentinel values with NaN.
- [ ] Parsed dates with explicit format and UTC.
- [ ] Stripped currency / unit decorations.
- [ ] Normalized booleans.
- [ ] Asserted schema at load time (and re-asserted after each transform).
