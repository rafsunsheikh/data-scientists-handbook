# Visualizing Comparisons

> **TL;DR** — Bars are the workhorse: sorted, horizontal, with one bar per category, labeled. Most other chart types ("dot plot", "lollipop", "dumbbell") are bar-variants optimized for specific comparison tasks.

## Pick a chart

| Comparison | Chart |
|---|---|
| Magnitudes of categories | **Bar (sorted, horizontal)** |
| Magnitudes, sparse | **Dot plot / lollipop** |
| Two values per category (before vs. after, A vs. B) | **Dumbbell** / paired dot |
| Magnitude change between two points in time | **Slope chart** |
| Magnitude across categories *and* a second dimension | **Heatmap** or **grouped bar** |
| Magnitudes that sum to a whole | See [compositions.md](compositions.md) |
| Magnitudes with uncertainty | **Bar + error bars**, or better, **dot + CI** |
| Many categories | **Pareto / waterfall** for cumulative contribution |

## Rules for bar charts

1. **Sort by value**, descending. Alphabetical order is almost never the right order.
2. **Horizontal** when category labels are long.
3. **Label values directly** if accuracy matters more than gestalt.
4. **Start at zero** — this is non-negotiable for bar charts (lengths must be comparable).
5. **Use one color** unless category color carries meaning.
6. **Annotate the conclusion** — e.g., highlight the bar that's the point of the chart.

```python
import seaborn as sns
import matplotlib.pyplot as plt

order = df.groupby('country')['revenue'].sum().sort_values(ascending=False).index
sns.barplot(df, x='revenue', y='country', order=order, color='steelblue')
plt.title('Revenue by country, FY24')
```

## Dot plot vs. bar

Dot plots use less ink for the same comparison. They're better than bars when:

- You have many categories (a long stack of bars is visually heavy).
- The values don't all start meaningfully at zero (e.g., country GDP per capita).
- You want to overlay a second value per category (paired dot / dumbbell).

## Dumbbell — before vs. after

```python
# Conceptual: one row per category, two dots connected by a line
```

Excellent for showing change without losing the *level*. Pair with sorting by the magnitude of the change.

## Slope chart — two time points

A line connecting two points (e.g., 2020 → 2024) per entity. Better than a bar pair for showing *change* directly.

## Grouped bar vs. stacked bar

- **Grouped** when the comparison of subcategories within a category is the point.
- **Stacked** when the total is the point and subcategory breakdown is secondary.
- **100% stacked** when the share (not the total) is the point.

If you have more than ~3 subcategories, a grouped bar gets unreadable — switch to small multiples (one chart per subcategory) or a heatmap.

## Heatmap for category × category

```python
ct = pd.crosstab(df['segment'], df['region'])
sns.heatmap(ct, annot=True, fmt='d', cmap='Blues')
```

For magnitude, use a sequential colormap. For deviation from a baseline, use a diverging colormap centered at zero.

## Showing uncertainty

Bars with error bars are the default but easy to misread. Prefer:

- **Dot + horizontal CI line** for confidence intervals.
- **Faded distribution** (gradient or hex bin) when the underlying distribution matters.
- **Multiple thin lines** for bootstrap replicates.

## Pitfalls

1. **Unsorted bars.** Alphabetical order destroys the comparison the chart is supposed to enable.
2. **Truncated y-axis** on a bar chart. Lengths must be comparable from zero.
3. **3-D bars** distort length perception.
4. **Pie when bar would work** — humans compare lengths better than angles.
5. **Too many colors.** A single color is fine if categories aren't a comparison.
6. **Dual y-axes** for two different units — almost always misleading; use small multiples.
