# Visualizing Compositions

> **TL;DR** — Composition is the "parts of a whole" question. Stacked bar and 100%-stacked bar handle most cases. Treemap and sunburst work for hierarchy. Sankey / alluvial work for flow. Pie charts work for ≤ ~5 categories *or* not at all.

## Pick a chart

| Goal | Chart |
|---|---|
| Single whole, few parts (≤5) | Pie / donut (just barely) — or 100% stacked bar |
| Single whole, many parts | Sorted bar, treemap |
| Hierarchical whole | Treemap, sunburst, icicle |
| Composition across categories (compare totals + shares) | Stacked bar |
| Composition across categories (compare shares only) | 100% stacked bar |
| Composition over time (compare totals + shares) | Stacked area |
| Composition over time (compare shares only) | 100% stacked area / `streamgraph` |
| Movement between categories | Sankey, alluvial |
| Funnel (sequential drop-off) | Funnel chart (or just a sorted horizontal bar) |
| Two compositions side-by-side | Mirrored stacked bar / "tornado" |

## Stacked bar — the default

```python
df.set_index('category')[['A', 'B', 'C', 'D']].plot.bar(stacked=True)
```

- Order the *segments* (subcategories) consistently across bars.
- Put the segment whose changes you care most about at the bottom (baseline = 0 makes it easiest to read).
- For more than ~5 segments, use small multiples (one chart per segment) instead.

## 100% stacked bar — compare shares only

When the *total* per category is irrelevant or confusing, normalize each bar to 100%. Excellent for comparing the mix across segments.

## Treemap — hierarchical wholes

```python
import squarify
squarify.plot(sizes=df['value'], label=df['name'], alpha=0.7)
```

Good for: budget breakdowns, market-cap comparisons, file-system maps.

Bad for: small categories (they shrink to unreadable rectangles) and any case where precise comparison matters (humans judge area poorly).

## Sunburst / icicle — multi-level hierarchy

Concentric or stacked rings, each level a deeper layer of the hierarchy.

```python
import plotly.express as px
px.sunburst(df, path=['region', 'country', 'city'], values='revenue').show()
```

Visually striking; people enjoy them; but for *measurement*, a treemap or nested bar is usually more readable.

## Sankey / alluvial — flow

When entities move between categories — say, user funnel stages, supply-chain flows, energy flows — Sankey makes the dominant flows obvious.

```python
import plotly.graph_objects as go
go.Figure(go.Sankey(node=dict(label=labels), link=dict(source=src, target=tgt, value=vals))).show()
```

Watch out: with many small flows, the diagram becomes a nest of lines. Group small flows into "Other" first.

## Pie chart — almost never

Use only when:

1. There are ≤ 5 slices.
2. The slices' relative sizes are very different.
3. Approximate proportion ("about half") is enough.

If you find yourself labeling each slice with the percentage, you've already conceded that the pie isn't doing the job. Use a sorted bar.

## Donut — same as pie

Same rules. The hole adds no information.

## Funnel — sequential composition

Standard for marketing / sales pipelines. A sorted horizontal bar with each bar narrower than the last works just as well and is usually more honest about the magnitudes.

## Mosaic plot — two categoricals

A rectangle subdivided by both categorical variables. Tile area = joint frequency. Good for spotting association.

```python
from statsmodels.graphics.mosaicplot import mosaic
mosaic(df, ['segment', 'region'])
```

## Streamgraph — compositional time series

A centered stacked area chart. Pretty; good for showing rise and fall over time. Hard to read individual layers in the middle of the stack — use small multiples if precise reading matters.

## Pitfalls

1. **More than 5 pie slices.** Switch to bar.
2. **Donut, pie, *and* 3-D.** Compounding bad choices.
3. **Inconsistent segment ordering** across stacked bars → impossible to compare.
4. **Stacked bar with negative values** (e.g., profit + loss) → use diverging bar instead.
5. **Streamgraph for precise reading.**
6. **Sankey with hundreds of nodes and edges.** Reduce or aggregate first.
7. **Treemap for time-series.** Treemaps are static snapshots.
