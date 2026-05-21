# Multivariate Visualization

> **TL;DR** — Past three variables, every chart type has trade-offs. The fundamental techniques are: encode additional variables with color/size/shape; use a scatter matrix; use small multiples (faceting); reduce to 2-D with PCA / UMAP / t-SNE; or use parallel coordinates for ordered numerical features.

## Pick a chart

| Variables | Chart |
|---|---|
| 3–4 numerical | Scatter + color/size + shape |
| 4–10 numerical | **Scatter matrix (pair plot)** |
| ≤ ~20 numerical, comparing entities | **Parallel coordinates** |
| Few entities, ≤ ~10 numerical | **Radar / spider** (use sparingly) |
| Numerical + categorical × categorical | **Faceted (small-multiple) plots** |
| Many numerical features, similar scale | **Heatmap** of values (entities × features) |
| High-dim → 2-D for clustering | **PCA / t-SNE / UMAP** |
| Many numerical, look at pairwise relationships | **Correlation heatmap** + **scatter matrix** |
| Categorical interactions | **Mosaic plot** |
| Multivariate time series | **Stacked small-multiples**, **heatmap** (rows = series, cols = time) |

## Scatter matrix (pair plot)

```python
sns.pairplot(df, hue='target', diag_kind='kde', plot_kws={'alpha': 0.6, 's': 10})
```

- Diagonal: marginal distribution per variable.
- Off-diagonal: scatter between pairs.
- `hue` adds a categorical dimension.

Works for ≤ ~8 variables. Past that, the grid becomes unreadable; switch to a correlation heatmap.

## Parallel coordinates

```python
from pandas.plotting import parallel_coordinates
parallel_coordinates(df, class_column='cluster', colormap='tab10', alpha=0.4)
```

Each row of data is a polyline crossing one vertical axis per feature. Patterns:

- Parallel lines → positive correlation.
- Crossing X → negative correlation.
- Bundles of similar lines → clusters.

Tips:

- **Reorder axes** so highly correlated features are adjacent (otherwise the chart is misleading).
- **Standardize** all variables to comparable scales.
- **Limit the number of lines** (sample, or aggregate to medians by cluster).

## Radar / spider chart — use sparingly

Useful for comparing a few entities across the same set of features (e.g., player stats, product specs). Issues:

- Area depends on axis order, which changes interpretation.
- Hard to read beyond ~6 axes.
- Bar or dot plots are usually more honest.

If you must use radar, fill polygons with low alpha and limit to 2–4 entities per chart.

## Heatmap of features

When you have many rows and many numerical columns on similar scales (e.g., gene-expression data), a heatmap is the right view:

```python
sns.clustermap(df, z_score=0, cmap='vlag', center=0)
```

- `z_score=0` standardizes rows.
- `cmap='vlag'` or `'coolwarm'` (diverging) for centered data.
- Clustermap reorders rows and columns by hierarchical clustering — interesting structure pops out.

## Faceting (small multiples)

The most underrated multivariate technique: split one chart into a grid by a categorical variable.

```python
sns.relplot(df, x='x', y='y', hue='c', col='segment', row='region', kind='scatter')
```

Each panel is identical except for the filter — the viewer's eye does the comparison.

## Dimensionality reduction

| Method | When | Notes |
|---|---|---|
| **PCA** | Variance-preserving, linear | Axes interpretable as loadings |
| **t-SNE** | Local structure, exploratory | Distort global distances |
| **UMAP** | Faster, more global structure than t-SNE | Sensitive to `n_neighbors` |
| **MDS / PCoA** | Distance matrix in, low-D out | Slow for large `n` |
| **LDA** (supervised) | Class separation in low-D | Linear |
| **Autoencoder bottleneck** | Learned non-linear | Deep-learning workflow |

```python
from sklearn.decomposition import PCA
pca = PCA(n_components=2).fit(X)
plt.scatter(pca.transform(X)[:, 0], pca.transform(X)[:, 1], c=y, cmap='tab10')

# Show how much variance each PC explains
plt.plot(np.cumsum(pca.explained_variance_ratio_))
```

Always show the explained-variance curve alongside the PC1-vs-PC2 scatter — it tells you whether 2 components are even reasonable.

## Pitfalls

1. **Reading t-SNE cluster sizes / inter-cluster distances** as meaningful. They aren't.
2. **PCA on data of wildly different scales** without standardizing — the largest-variance column dominates.
3. **Radar charts** with axes in different units.
4. **Parallel coordinates with axes in arbitrary order** — the visual story changes.
5. **Too many faceted panels** (>20) — viewers can't process.
6. **3-D scatter plots.** Rotation makes them seem informative; for static reports they hide structure.
