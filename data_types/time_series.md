# Time Series Data

> **TL;DR** — Time-series data is any sequence indexed by time. The structure of that index — *regular vs. irregular*, *univariate vs. multivariate*, *cross-sectional vs. panel*, *evenly-spaced events vs. timestamped logs* — drives the entire analysis. Most "weird" time-series bugs come from getting the index structure wrong.

## 1. Sub-types

| Sub-type | Description | Example |
|---|---|---|
| Regular univariate | Single variable at fixed intervals | hourly temperature |
| Irregular univariate | Single variable at arbitrary timestamps | server error log |
| Multivariate | Multiple variables at same timestamps | OHLCV stock data |
| Panel / longitudinal | Same variables tracked across many entities over time | sales by store by day |
| Event / point process | Timestamps of discrete events | click stream, earthquakes |
| Spatio-temporal | Time × location | weather grids, traffic |
| Hierarchical / grouped | Aggregates roll up | country/region/store sales |

## 2. The index is everything

```python
df.index = pd.to_datetime(df['timestamp'])
df = df.sort_index()
```

Common index issues:

- **Mixed timezones.** Store everything in UTC internally; convert to local time only for display.
- **DST jumps.** Twice a year a clock skips or repeats an hour. Naive timestamps lose 1 hour of data or duplicate it.
- **Leap seconds.** Rare but real for high-frequency financial data.
- **Off-by-one in resampling.** `'D'` is calendar day, `'24H'` is rolling 24-hour. Different answers near DST.
- **Unsorted index.** Many pandas ops silently misbehave (or are slow) on unsorted timestamps.

## 3. Common transformations

- **Resampling** — change frequency: `df.resample('1H').mean()`.
- **Rolling window** — moving averages, rolling std: `df.rolling(7).mean()`.
- **Lagging** — `df['lag1'] = df['x'].shift(1)`. Essential for forecasting features. Be careful: shift then split, never the reverse.
- **Differencing** — `df['x'].diff()` for stationarity.
- **Decomposition** — trend + seasonality + residual (STL is robust).

## 4. Properties to characterize

| Property | How to detect | Why it matters |
|---|---|---|
| Stationarity | ADF / KPSS test | Many models assume it |
| Trend | Visual; STL decomposition | Detrend before some models |
| Seasonality | ACF, periodogram, STL | Pick seasonal periods |
| Autocorrelation | ACF / PACF plots | Choose AR/MA lags |
| Cycles | Spectral analysis | Distinguish from seasonality |
| Structural breaks | Chow test, CUSUM | Models trained pre-break may fail post-break |
| Missing timestamps | Gap detection on the index | Most libraries assume continuous index |
| Outliers / level shifts | Visual + Hampel filter | Single events can dominate the model |

## 5. Common pitfalls

1. **Train/test leakage.** Random k-fold is wrong for time series — use time-based splits or rolling-origin evaluation.
2. **Look-ahead in features.** Using future information (e.g., a 30-day average that includes the target day) silently inflates accuracy.
3. **Forecasting on a non-stationary series.** Many classical models require differencing first.
4. **Ignoring weekends/holidays.** Daily retail data has a strong weekly cycle and big holiday effects.
5. **Forgetting timezone.** "Daily" sales in a global company need a chosen anchor (UTC? store-local?).
6. **Resample without specifying `closed`/`label`.** Boundaries silently shift.
7. **Outlier detection that mistakes seasonality for anomalies.** Decompose first.

## 6. Cleaning checklist

See [`data_cleaning/time_series_cleaning.md`](../data_cleaning/time_series_cleaning.md).

- [ ] Parse and standardize timezone (prefer UTC).
- [ ] Sort by index.
- [ ] Detect duplicates and decide aggregation rule.
- [ ] Detect gaps; decide imputation (forward fill, interpolation, model-based).
- [ ] Decompose to confirm trend / seasonality before further modeling.
- [ ] Plot the series — eyeballing catches things tests miss.

## 7. Visualizations

| Question | Chart |
|---|---|
| What does the series look like? | Line chart |
| What is the trend? | Line + LOWESS / rolling mean |
| Is there seasonality? | Seasonal subseries plot; STL decomposition |
| How autocorrelated is it? | ACF / PACF |
| Where is the spectral energy? | Periodogram |
| Many series at once? | Small multiples (faceted lines), heatmap (rows = entity, cols = time) |
| Event data? | Event raster plot, histogram of inter-arrival times |

See [`data_visualization/trends.md`](../data_visualization/trends.md).

## 8. Models to know

| Model class | Use when |
|---|---|
| Naive (last value, seasonal naive) | Baseline — always start here |
| Exponential smoothing (ETS, Holt-Winters) | Short, low-noise, seasonal |
| ARIMA / SARIMA | Stationary or differenced, well-studied series |
| Prophet | Daily business series with holidays and trend changes |
| State-space / Kalman filter | Online updates, missing data |
| Gradient boosting on lag features | Mixed regressors, panel data |
| RNN / Transformer (Temporal Fusion Transformer, N-BEATS, TimesFM) | Large-scale, high-dimensional, learned representations |

## 9. References

- Hyndman & Athanasopoulos. *Forecasting: Principles and Practice* (3rd ed., free online).
- Box, Jenkins, Reinsel, Ljung. *Time Series Analysis: Forecasting and Control*.
- Hamilton. *Time Series Analysis*.
