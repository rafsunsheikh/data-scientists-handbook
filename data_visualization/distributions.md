# Visualizing Distributions

> **TL;DR** — Use a histogram for first inspection, a KDE for a smooth view, an ECDF when you need to read quantiles directly, and a box / violin / strip when comparing across groups. Avoid pie charts; avoid `bar` for what should be `hist`.

## When to use which

| Goal | Best chart |
|---|---|
| Quick shape, with bins | **Histogram** |
| Smooth shape, fewer parameters | **KDE** |
| Exact quantiles and percentiles, no binning | **ECDF (CDF)** |
| Compare distribution across groups (compact) | **Box plot** |
| Compare distribution across groups (richer) | **Violin plot** |
| Show all individual points across groups | **Strip / swarm plot** |
| Many groups stacked vertically | **Ridge plot** |
| Check normality | **Q-Q plot** |
| Heavy-tailed shape, log scale | **Log-x histogram** or **ECDF on log axis** |
| Two distributions overlaid | **Overlapping KDE** or **histogram with `alpha`** |

## Reference: matplotlib / seaborn

```python
import seaborn as sns
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 4, figsize=(16, 4))
sns.histplot(df, x='income', bins=50, ax=axes[0]);   axes[0].set_title('Histogram')
sns.kdeplot(df, x='income', ax=axes[1]);              axes[1].set_title('KDE')
sns.ecdfplot(df, x='income', ax=axes[2]);             axes[2].set_title('ECDF')
sns.boxplot(df, y='income', x='segment', ax=axes[3]); axes[3].set_title('Box by group')
plt.tight_layout()
```

## Histogram — choosing bins

The bin choice changes the shape:

- Too few bins → over-smoothed, hides modes.
- Too many bins → noisy.
- Defaults: Sturges, Freedman-Diaconis (`'fd'` in `np.histogram_bin_edges`), or Scott's rule. Freedman-Diaconis is the most robust default.

```python
plt.hist(x, bins='fd')
```

For wildly skewed data, use a log x-axis (`plt.xscale('log')`) or log-transform the data.

## KDE — be aware of the bandwidth

The bandwidth (smoothness) parameter is to KDE what bin width is to histogram. seaborn picks a reasonable default (Scott's rule), but for bimodal or heavy-tailed data, the default oversmooths. Override with `bw_adjust`:

```python
sns.kdeplot(df, x='x', bw_adjust=0.5)
```

KDE assumes the data is continuous and infinite — bounded data (e.g., probabilities in `[0,1]`) often shows a fake tail past the boundary. Truncate the plot or use `cut=0`.

## ECDF — most underused

The empirical CDF is binning-free, makes the median and percentiles literally readable, and overlays cleanly for comparing groups.

```python
sns.ecdfplot(df, x='income', hue='segment')
```

If you only learn one new chart from this section, learn the ECDF.

## Box plot — the standard, with caveats

- The box is Q1 to Q3 (IQR).
- The "whiskers" extend to the most extreme point within 1.5×IQR of the box.
- Anything outside is plotted as an outlier (Tukey, 1977).

Caveats:

- Hides bimodality (a U-shaped distribution and a normal look identical).
- For small `n`, the box is misleading. Use a strip plot too.

## Violin plot — box + KDE

Use when you want both the summary (box-like) and the shape (KDE) per group.

```python
sns.violinplot(df, x='segment', y='income', inner='box')
```

For small `n`, prefer **strip** or **swarm** to show every point.

## Q-Q plot — assess distribution fit

```python
import scipy.stats as stats
stats.probplot(x, dist='norm', plot=plt.gca())
```

Straight line → fits the reference distribution. Departures at the tails reveal heavy tails or skew.

## Comparing two distributions

| Goal | Chart |
|---|---|
| Eyeball the difference | Overlapping KDE; histograms with `alpha` |
| Quantify the difference | ECDFs for each group; vertical gap shows distributional shift |
| Test the difference | KS test on the two ECDFs |

## Pitfalls

1. **Binned histogram with too few bins** hides modes.
2. **Mean ± SD when the data is skewed** — show median + IQR or the full ECDF.
3. **Box plots with tiny `n`** — show individual points instead.
4. **Pie charts** with more than 5 categories — humans can't compare angles.
5. **Bar chart of values** when you meant a histogram.
6. **Truncated y-axis on a bar chart** — exaggerates differences. (On a line chart, sometimes fine.)
7. **Plotting two distributions on different scales side-by-side** — share the x-axis.

## Companion notebook

[`../notebooks/visualization/01_distributions.ipynb`](../notebooks/visualization/01_distributions.ipynb)
