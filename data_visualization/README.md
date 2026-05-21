# Data Visualization

> **TL;DR** — Pick the chart based on (a) the **purpose** of the comparison you want the viewer to make and (b) the **types** of the data you're showing. Get those two right and you'll rarely pick the wrong chart. Get either wrong and even a beautiful chart is misleading.

## A picker: purpose × data type

| Purpose | One numerical | Two numerical | Numerical × categorical | Two categorical | Time series | Geospatial | Text | Network |
|---|---|---|---|---|---|---|---|---|
| **Distribution** | Histogram, KDE, ECDF, box, violin | 2D hex / KDE | Box, strip, violin by group | Mosaic | — | Choropleth, hexbin map | — | Degree histogram |
| **Comparison** | — | Slope chart | Bar, dot plot, dumbbell | Heatmap of cross-tab | Multiple lines | Small-multiple maps | Frequency bar | Adjacency-matrix heatmap |
| **Relationship** | — | Scatter, hexbin, 2D KDE | Box, violin | Mosaic, heatmap | Cross-correlation | Bivariate choropleth | Cosine sim heatmap | Force-directed layout |
| **Composition** | — | — | Stacked bar | Stacked / 100% bar, treemap | Stacked area | — | Topic bars | Sankey, alluvial |
| **Trend** | — | — | — | — | Line, area | Animated map | Trend of term over time | Network growth animation |
| **Ranking** | — | — | Sorted bar, dot plot, lollipop | — | Line, bump chart | Symbol map | Top-N words | Centrality bar |

## Chapter index

| Chapter | Covers |
|---|---|
| [distributions.md](distributions.md) | Histogram, KDE, ECDF, box, violin, ridge, q-q |
| [comparisons.md](comparisons.md) | Bar, dot, dumbbell, slope, lollipop |
| [relationships.md](relationships.md) | Scatter, bubble, hexbin, 2D-KDE, correlation matrix |
| [compositions.md](compositions.md) | Stacked bar, treemap, sunburst, Sankey, alluvial |
| [trends.md](trends.md) | Line, area, candlestick, decomposition, seasonal subseries |
| [geospatial_viz.md](geospatial_viz.md) | Choropleth, hexbin map, flow, 3-D, tiles |
| [network_viz.md](network_viz.md) | Force layouts, arc, matrix, community coloring |
| [text_viz.md](text_viz.md) | Word frequencies, embeddings, topic models |
| [image_viz.md](image_viz.md) | Image grids, confusion matrices with samples, Grad-CAM |
| [multivariate.md](multivariate.md) | Parallel coords, radar, PCP, scatter matrix, t-SNE, UMAP |
| [dashboards_and_storytelling.md](dashboards_and_storytelling.md) | When charts become a narrative |

## Principles to apply to every chart

1. **Start from the question, not the chart.** "What proportion of revenue comes from the top 5 customers?" → sorted bar. "How has churn evolved week-over-week?" → line.
2. **Encode quantitative data with position before length, area, or color.** Position is the most perceptually accurate channel (Cleveland & McGill, 1984 [^cm]).
3. **Tell viewers what they're looking at.** Title states the conclusion, not the variables. Subtitle states the data source and date. Axes are labeled with units.
4. **Annotate the point.** Highlight the bar / point / region that matters. Don't make the reader hunt.
5. **Use color with intent.** Categorical palette for categories; sequential for ordered magnitude; diverging only when there's a meaningful midpoint. Color-blind safe by default (`viridis`, `cividis`).
6. **Sort by what matters.** Default bar order is alphabetical; almost always sort by value instead.
7. **Use small multiples instead of overplotting.** A grid of small charts beats one busy chart for >3 groups.
8. **Show the data, not just summaries.** Strip and dot plots reveal what bars hide.
9. **Be honest about axes.** Truncated y-axes on bar charts exaggerate differences; truncated y on line charts can be appropriate. Document the choice.
10. **No 3-D, no pie charts >5 slices, no dual axes** unless you have a specific defensible reason.

## Libraries (Python)

| Library | Strengths | Notes |
|---|---|---|
| `matplotlib` | Most flexible; production-grade | Verbose; the foundation under most others |
| `seaborn` | Statistical defaults; great EDA | Layered on matplotlib |
| `plotly` | Interactive; web-friendly | Heavier; great for dashboards |
| `altair` | Grammar-of-graphics; declarative | Limited data size unless using vega-fusion |
| `bokeh` | Interactive; server apps | Mature alternative to plotly |
| `plotnine` | ggplot2 grammar | Familiar if you come from R |
| `holoviews` / `hvplot` | High-level wrapper for many backends | Quick EDA at scale |
| `datashader` | Renders billions of points | When `plotly` and `matplotlib` choke |

For static reports, prefer matplotlib/seaborn. For interactive exploration, plotly or altair. For dashboards, see [dashboards_and_storytelling.md](dashboards_and_storytelling.md).

## Companion notebooks

- [`../notebooks/visualization/01_distributions.ipynb`](../notebooks/visualization/01_distributions.ipynb)
- [`../notebooks/visualization/02_relationships.ipynb`](../notebooks/visualization/02_relationships.ipynb)
- [`../notebooks/visualization/03_trends.ipynb`](../notebooks/visualization/03_trends.ipynb)

[^cm]: Cleveland, W. S. & McGill, R. (1984). *Graphical Perception: Theory, Experimentation, and Application to the Development of Graphical Methods.* JASA 79(387).
