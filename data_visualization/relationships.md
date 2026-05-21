# Visualizing Relationships

> **TL;DR** — Scatter is the default for two numerical variables. Once you have more than ~5,000 points, overplotting hides the structure — switch to hexbin or 2D-KDE. For more than two variables at once, use a correlation matrix, scatter matrix, or dimensionality reduction.

## Pick a chart

| Variables | Sample size | Chart |
|---|---|---|
| 2 numerical | <5k | **Scatter** |
| 2 numerical | 5k–500k | **Hexbin** or **2D-KDE** |
| 2 numerical | >500k | **Datashader** raster |
| 2 numerical, time-ordered | any | **Connected scatter** or **lineplot** |
| 2 numerical + 1 categorical | <5k | Scatter with color |
| 2 numerical + 1 numerical | <5k | **Bubble** (size = 3rd var) — use sparingly |
| Many numerical | any | **Correlation heatmap**, **scatter matrix** |
| Many numerical, high dim | any | **PCA / t-SNE / UMAP** |
| Numerical × categorical | — | See [distributions.md](distributions.md) |

## Scatter

```python
sns.scatterplot(df, x='income', y='spend', hue='segment', alpha=0.6)
```

- Use `alpha < 1` once you have hundreds of points.
- Add a **regression line** (`sns.regplot` / `sns.lmplot`) only if linearity is plausible — otherwise it lies.
- Always check the axes for outliers that compress the bulk of the data into a corner. Consider log scales.

## Hexbin / 2D-KDE — when scatter clogs

Once you can't see individual points through the cloud, count is what matters:

```python
plt.hexbin(df['x'], df['y'], gridsize=40, cmap='viridis')
plt.colorbar(label='count')
```

2D-KDE produces a smooth contour map; hexbin is more honest about where the data actually lives.

## Datashader — millions of points

```python
import datashader as ds
canvas = ds.Canvas(plot_width=800, plot_height=600)
agg = canvas.points(df, 'x', 'y')
```

Renders billions of points to a pixel grid in seconds; integrate with hvplot or plotly for interactivity.

## Correlation matrix

```python
corr = df.select_dtypes('number').corr()
sns.heatmap(corr, annot=True, cmap='coolwarm', center=0, vmin=-1, vmax=1, fmt='.2f')
```

- Use a **diverging** colormap centered at zero.
- Cluster rows / columns to bring correlated groups together (`sns.clustermap`).
- Pearson assumes linearity; for monotonic non-linear use Spearman; for ties / ordinal use Kendall.
- Beware: high correlation ≠ causation, and zero correlation ≠ independence (think `y = x²`).

## Scatter matrix (pair plot)

```python
sns.pairplot(df, hue='segment', diag_kind='kde', plot_kws={'alpha': 0.5})
```

Great for EDA on ≤ ~10 variables. Past that it becomes too dense.

## Bubble — be careful

Mapping a third variable to point *size* is appealing but perceptually weak (humans badly judge area). If you do:

- Size area, not radius.
- Scale to a reasonable range; bubbles shouldn't overlap heavily.
- Prefer color (with a sequential colormap) over size when possible.

## Time-ordered relationships

When (x, y) points have a time order (e.g., GDP vs. inequality over years), use:

- **Connected scatter** — scatter with a line connecting consecutive years, optionally with arrowheads.
- **Phase portrait** — `(x_t, x_{t+1})` to inspect dynamics.

## High-dimensional structure

| Method | Strength | Weakness |
|---|---|---|
| **PCA** | Linear, fast, interpretable axes | Misses non-linear structure |
| **t-SNE** | Preserves local structure | Global distances meaningless; non-deterministic |
| **UMAP** | Faster than t-SNE; preserves more global structure | Sensitive to `n_neighbors` |
| **PaCMAP / TriMap** | Better global preservation | Less common |
| **PCoA / MDS** | Works on distance matrix directly | Slow for large `n` |

**For all non-linear methods**, multiple runs give different layouts. Don't read fine geometry as if it were ground truth — only cluster structure.

```python
from umap import UMAP
emb = UMAP(n_components=2, n_neighbors=30, min_dist=0.1).fit_transform(X)
plt.scatter(emb[:, 0], emb[:, 1], c=labels, cmap='tab20', s=4, alpha=0.7)
```

## Pitfalls

1. **Overplotting** that hides the structure you're trying to see.
2. **Truncated axes** that exaggerate or hide trends.
3. **Reading t-SNE cluster sizes / distances** as if they were faithful — they're not.
4. **Correlation interpreted as causation.**
5. **Spurious correlations** from short time series (Tyler Vigen's catalog).
6. **Bubble charts with three encoded variables** that no one can read.
7. **Regression lines on non-linear data.**

## Companion notebook

[`../notebooks/visualization/02_relationships.ipynb`](../notebooks/visualization/02_relationships.ipynb)
