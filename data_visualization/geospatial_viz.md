# Geospatial Visualization

> **TL;DR** — Choose your map type by what you're encoding. Counts per region → choropleth (in an equal-area projection). Point density → hexbin / H3 map. Flows → arrow / Sankey-on-map. Always show a scale, a legend, and the projection.

## Pick a map

| Encoding | Map type |
|---|---|
| One value per region (rate, density) | **Choropleth** |
| Magnitude at point locations | **Graduated symbol** |
| Density of point events | **Heatmap / KDE** or **hexbin** / **H3 cells** |
| Counts that need to be honest about area | **Dot density** (1 dot = N items) |
| Movement A → B | **Flow / arrow / line map**, optionally a Sankey overlay |
| 3-D magnitude | **Extruded prism** (deck.gl, kepler.gl) |
| Routes | **Polyline overlay** |
| Time-varying | **Animation** or **small-multiple time slices** |
| Multi-variate | **Bivariate choropleth** (2-variable color grid) |

## Choropleth — and the projection problem

```python
import geopandas as gpd
ax = gdf.to_crs(epsg=3035).plot(   # Lambert equal-area, Europe
    column='rate', legend=True, cmap='viridis', edgecolor='white', linewidth=0.3)
ax.set_axis_off()
```

- **Use an equal-area projection** (e.g., Albers, Lambert) for choropleths. Web Mercator distorts area; on a global map, Greenland looks bigger than Africa.
- **Normalize by area or population.** Raw counts visualized as choropleth just show "where lots of people live."
- **Use a sequential colormap.** Diverging only when there's a meaningful midpoint (e.g., percent change > 0 vs. < 0).
- **Bin the colors** (`pysal.viz.mapclassify`) — `Quantiles`, `NaturalBreaks`, `EqualInterval`. Different binning tells different stories; choose deliberately and document.

## Bivariate choropleth

Show two variables on a single map by mapping color to a 2-D palette (e.g., 3×3 grid):

```
       low income → high income
low risk    LL  ML  HL
              vs.
high risk   LM  MM  HM
              vs.
            LH  MH  HH
```

Powerful for "where are both X and Y high?" — but the legend is its own art and most viewers need an explanation.

## Hexbin / H3

For point data, aggregating to a regular grid is more honest than choropleth-by-region (regions of unequal size bias the visual). H3 (Uber's hexagonal hierarchical index) is increasingly standard.

```python
import h3
df['h3'] = df.apply(lambda r: h3.latlng_to_cell(r['lat'], r['lng'], 7), axis=1)
counts = df.groupby('h3').size().reset_index(name='n')
```

## Heatmap / KDE

A smoothed density of points. Easy to read but the bandwidth choice is meaningful — too small shows blobs, too large shows nothing. State the bandwidth in the caption.

## Dot density

One dot per N events; dots randomly jittered within the polygon. Avoids the "region size confuses magnitude" problem. Tradeoff: very precise locations are *not* shown — be honest about the jitter.

## Flow maps

For migrations, trade flows, transit. Issues:

- Lines crossing the dateline.
- Asymmetric flows (`A→B` vs. `B→A`).
- Visual clutter — bundle edges, threshold small flows.

Tools: kepler.gl, deck.gl, mapbox, plotly (`scattergeo` with `lines`), matplotlib + `cartopy`.

## Tile-based maps

Add a basemap (street, satellite) for context, especially at city scale:

```python
import contextily as cx
ax = gdf.to_crs(epsg=3857).plot(column='value', alpha=0.7)
cx.add_basemap(ax, source=cx.providers.CartoDB.Positron)
```

Pick a low-contrast basemap (CartoDB Positron, Stamen Toner Lite) so your data stands out.

## Static vs. interactive

| Tool | When |
|---|---|
| `matplotlib` + `geopandas` + `contextily` | Static reports, papers |
| `folium` (Leaflet) | Quick interactive notebooks |
| `plotly` `scattermapbox` / `choroplethmapbox` | Interactive with Mapbox tiles |
| `kepler.gl` / `deck.gl` | Large datasets, 3-D, web |
| `geemap` | Earth Engine integration |
| `lonboard` | Very large point clouds in notebooks |

## Always include

- **Title** with the conclusion.
- **Legend** with units and bin definitions.
- **Scale bar** (especially at city/region scale).
- **North arrow** if the map is rotated.
- **Source and date.**
- **Projection** stated, when it matters (e.g., when comparing areas).

## Pitfalls

1. **Choropleth in Web Mercator** distorts area.
2. **Raw counts on a population-uneven base.** Always normalize.
3. **Diverging colormap with no midpoint.**
4. **Too many bins** (rainbow palette) — viewers can't decode 10 colors.
5. **Pie chart on every region** ("pie-chart map") — universally bad.
6. **Hiding the data by overplotting** — many points stacked on the same spot.
7. **Forgetting to project before computing distance / area.** (See [`../data_cleaning/geospatial_cleaning.md`](../data_cleaning/geospatial_cleaning.md).)

## References

- Brewer. *Designing Better Maps.*
- ColorBrewer — https://colorbrewer2.org (color palettes for maps).
- Lovelace et al. *Geocomputation with R.*
