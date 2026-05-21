# Time Series Cleaning

> **TL;DR** — Time-series cleaning is mostly about fixing the *index*: timezone, sort order, duplicates at the same timestamp, gaps in coverage. Get the index right and most downstream tools work. Get it wrong and they all silently misbehave.

## 1. Parse the timestamp correctly

```python
df['ts'] = pd.to_datetime(df['ts'], utc=True, format='ISO8601', errors='coerce')
df = df.dropna(subset=['ts']).set_index('ts').sort_index()
```

- **Always UTC** internally. Convert for display only.
- **Specify format** when possible to avoid ambiguous parses.
- **Sort the index**, otherwise rolling / resample misbehave.

## 2. Detect and handle duplicates

```python
df.index.duplicated().sum()
df = df.groupby(level=0).mean()   # or .last(), .sum() — depending on semantics
```

For event data, duplicates at the same timestamp may be legitimate (multiple events in the same millisecond). Decide the rule explicitly.

## 3. Detect gaps

```python
expected = pd.date_range(df.index.min(), df.index.max(), freq='1H')
missing = expected.difference(df.index)
len(missing), missing[:10]
```

For regular series, reindex to fill in the expected timestamps:

```python
df = df.reindex(expected)
```

`NaN` rows then represent the gaps, which can be filled.

## 4. Filling gaps

| Method | When |
|---|---|
| `ffill` (last observation carried forward) | Slow-changing state (e.g., daily price) |
| `bfill` | Symmetric to ffill; rarely used alone |
| `interpolate('linear')` | Smooth signals |
| `interpolate('time')` | Irregular index, accounts for spacing |
| `interpolate('spline', order=3)` | Smooth curvature |
| Seasonal-naïve | Strong weekly / yearly pattern: copy from previous period |
| STL decompose + interpolate residuals | Decompose, interpolate residual, recompose |
| Kalman smoother | Principled, gives uncertainty |
| Domain-specific (e.g., `0` for "stockout") | When zero is the truth |

Never blindly forward-fill across very long gaps — it creates fake flatness.

## 5. Resampling

```python
df.resample('1H').mean()          # downsample with mean
df.resample('1H').agg({'price': 'last', 'volume': 'sum'})   # mixed rules
df.resample('15min').ffill()      # upsample with hold
```

Be explicit about `closed` and `label` — defaults differ between resample frequencies. For "1 hour ending at 14:00" use `closed='right', label='right'`.

## 6. Timezones and DST

```python
df.index.tz                       # confirm
df.index = df.index.tz_convert('Europe/Berlin')
```

DST jumps create gaps in spring and duplicates in autumn for local-time series. Either store in UTC, or use `ambiguous` and `nonexistent` parameters explicitly:

```python
df.index = df.index.tz_localize('Europe/Berlin',
                                ambiguous='NaT', nonexistent='shift_forward')
```

## 7. Outliers in time series

Distinct from i.i.d. outliers — context matters:

- **Additive outliers** — single anomalous point.
- **Level shifts** — sustained change in mean (regime change, sensor recalibration).
- **Innovation outliers** — affect all subsequent points (rare).

Tools: STL residuals + threshold, Hampel filter (rolling median + MAD), `ruptures` for change points, `stumpy` matrix profile for anomalous subsequences.

## 8. Re-aligning multivariate series

Different series may have different timestamps. Two strategies:

**Strategy A: union of timestamps + fill**

```python
joined = pd.concat([s1, s2], axis=1).sort_index().ffill()
```

**Strategy B: as-of join** (use the most recent value from B at each timestamp in A)

```python
pd.merge_asof(a.sort_index(), b.sort_index(),
              left_index=True, right_index=True, direction='backward')
```

As-of joins are the standard in financial / trading data.

## 9. Checklist

- [ ] Parsed and converted timestamps to UTC.
- [ ] Sorted by index.
- [ ] Handled duplicate timestamps with explicit aggregation rule.
- [ ] Detected gaps; filled where appropriate; left as NaN where not.
- [ ] Confirmed timezone for downstream consumers.
- [ ] Inspected for level shifts and structural breaks.
- [ ] Plotted the cleaned series — eyes catch what tests don't.
