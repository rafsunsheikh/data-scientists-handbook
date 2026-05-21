# Visualizing Trends Over Time

> **TL;DR** — A simple line chart with the time axis on the x-axis is the right answer for 90% of time-series visualizations. Add a smoothed trend if the data is noisy. For decomposition, use `STL` and plot trend / seasonal / residual separately. Avoid dual-axis charts.

## Pick a chart

| Goal | Chart |
|---|---|
| One series over time | **Line** |
| 2–5 series, comparable magnitudes | **Multiple lines** (one color each) |
| 2–5 series, different magnitudes | **Small multiples** (one chart each) |
| Many series (10+) | **Small multiples**, or **spaghetti with one highlighted** |
| Cumulative over time | **Area** (filled below the line) |
| Composition over time | **Stacked area** (see compositions.md) |
| Range / band over time | **Line + shaded band** |
| Financial OHLC | **Candlestick** |
| Trend vs. noise | **Line + LOWESS / rolling mean** |
| Annual cycle visible? | **Seasonal subseries plot** |
| Trend / seasonal / residual | **STL decomposition** (4-panel) |
| Lag / persistence | **ACF / PACF** |
| Spectral content | **Periodogram / spectrogram** |
| Event timestamps | **Event raster** or **histogram of times** |
| Calendar pattern | **Calendar heatmap** |

## Line chart — defaults that matter

```python
import matplotlib.pyplot as plt
df['x'].plot(figsize=(10, 4))
plt.title('Daily active users, 2024-01 to 2024-12')
plt.ylabel('users')
plt.grid(True, alpha=0.3)
```

- Time on x-axis, value on y-axis. Always.
- Y-axis can be truncated for line charts (unlike bars), but mark it clearly if it's a deliberate zoom.
- Use a light gridline; remove the top/right spines.
- For noisy data, overlay a 7-day or 30-day rolling mean.

## Many series

Two failure modes:

- **Spaghetti** — too many overlapping lines.
- **Crayon box** — too many colors, all similar.

Fixes:

- **Highlight one** line (color it), gray out the rest.
- **Small multiples** — one panel per series.
- **Aggregate** — show the median + IQR band of the group, not individual lines.

```python
fig, axes = plt.subplots(2, 3, figsize=(15, 8), sharex=True, sharey=True)
for ax, country in zip(axes.flat, top6):
    df[df['country'] == country]['value'].plot(ax=ax)
    ax.set_title(country)
```

## Showing uncertainty

Forecasts, confidence intervals, bootstrap bands:

```python
ax.plot(t, forecast, color='C0')
ax.fill_between(t, lo, hi, alpha=0.2, color='C0')
```

For multiple forecasts: stack the fan with progressively lighter shades.

## Seasonality

**Seasonal subseries plot** — one panel per period (e.g., 12 panels for monthly), each showing the within-period trend across years. Reveals which months are growing fastest.

**STL decomposition** — separate trend, seasonal, and residual:

```python
from statsmodels.tsa.seasonal import STL
res = STL(df['x'], period=7).fit()
res.plot()
```

## Autocorrelation / PACF

```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
plot_acf(df['x'], lags=40)
plot_pacf(df['x'], lags=40)
```

ACF that decays slowly → trend not removed.
ACF with sharp spikes at periodic lags → seasonality.

## Calendar heatmap

A grid with one cell per day, rows = weeks, columns = day-of-week. Reveals weekly patterns and outlier days at a glance.

Library: `calmap`, or manual matplotlib.

## Event raster

For point-process data (clicks, earthquakes, network events): one row per entity, dots at the timestamps.

```python
plt.eventplot(times_per_user, orientation='horizontal')
```

## Don't do this

1. **Dual axes** with two units. Almost always misleading. Use small multiples or normalize.
2. **Bar charts for time series.** Bars are for categorical comparison. Lines for ordered.
3. **Connecting non-equally-spaced points without a gap.** A line between Dec 31 and the next data point in June implies continuity that isn't there.
4. **Aggregating to a single value** when the variability is the story.
5. **Annotating every quarter** until the chart is unreadable. Annotate one moment that matters.
6. **3-D ribbon charts.** Same problem as 3-D anything.

## Companion notebook

[`../notebooks/visualization/03_trends.ipynb`](../notebooks/visualization/03_trends.ipynb)
