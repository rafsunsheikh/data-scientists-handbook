# Geospatial Data

> **TL;DR** — Geospatial data is data with a *location* attached, but "location" means many things: a point, a line, a polygon, a raster cell, or a trajectory. Two pieces of metadata determine whether your maps and joins are correct: the **coordinate reference system (CRS)** and the **geometry type**. Get either wrong and your map will be a kilometer (or a thousand) off.

## 1. Sub-types

### 1.1 Vector
| Geometry | Example |
|---|---|
| Point | a store, a tweet, a sensor |
| MultiPoint | constellation of locations |
| LineString | a road segment, a flight path |
| MultiLineString | a road network as one feature |
| Polygon | a building footprint, a census tract |
| MultiPolygon | a country with islands |
| GeometryCollection | mixed geometries in one feature |

### 1.2 Raster
A regular grid of cells with values (elevation, land cover, NDVI, population density).

### 1.3 Trajectories
Time-stamped sequences of points — GPS traces, ship AIS, drone flight logs. (Cross-reference: [time series](time_series.md).)

### 1.4 Networks
Topological graphs with geographic embedding — road networks, transit systems, utilities. (Cross-reference: [graph](graph.md).)

### 1.5 Point clouds
Unstructured 3-D points, typically from LiDAR.

## 2. Coordinate Reference Systems (CRS)

A CRS tells the computer how to map numeric coordinates to positions on Earth.

| CRS | EPSG | Used for | Notes |
|---|---|---|---|
| WGS 84 | 4326 | Lat/lon, GPS, GeoJSON default | Degrees, **not** meters |
| Web Mercator | 3857 | Web maps (Google/OSM/Mapbox tiles) | Distorts area away from equator |
| UTM zones | 32601–32660, 32701–32760 | Local distance / area work | Each zone is 6° wide |
| National grids (e.g., BNG, NAD83) | varies | Country-level work | Best accuracy locally |
| Equal-area (Mollweide, Albers) | varies | Choropleth comparisons | Honest area depiction |

Rules of thumb:

- Store data in WGS 84 (EPSG:4326) for portability.
- Reproject to a *projected* CRS (UTM, national grid) before computing distance, area, or buffer in meters.
- Reproject to an equal-area projection before making choropleth maps that compare regions.

## 3. Storage formats

| Format | Vector / raster | Notes |
|---|---|---|
| Shapefile (`.shp + .dbf + .shx + .prj`) | Vector | Legacy; 2GB limit; multi-file; column names truncated |
| GeoJSON | Vector | Web-friendly; large for big datasets |
| GeoPackage (`.gpkg`) | Both | Modern SQLite-based; preferred default |
| FlatGeobuf | Vector | Efficient streaming |
| Parquet + WKB | Vector | Columnar, integrates with data lakes |
| GeoTIFF | Raster | TIFF + georeferencing metadata |
| Zarr / COG (Cloud-Optimized GeoTIFF) | Raster | Streamable from object storage |
| NetCDF / HDF5 | Raster | Multidimensional climate / ocean data |
| LAS / LAZ | Point cloud | LiDAR |

## 4. Common pitfalls

1. **Lat/lon swapped.** GeoJSON is `[lon, lat]`; humans usually say "lat, lon"; many libraries differ. Always check.
2. **Computing distance in degrees.** A degree of longitude varies from ~111 km at the equator to 0 km at the poles. Reproject first.
3. **Missing or wrong CRS.** Most libraries assume EPSG:4326 silently. If you didn't set it explicitly, assume nothing.
4. **Invalid geometries.** Self-intersecting polygons, slivers, duplicate vertices. Run `make_valid` / `buffer(0)` before spatial joins.
5. **Spatial joins on huge data without an index.** Use STRtree / R-tree (`gpd.sjoin` uses one by default).
6. **Cartogram lies via Mercator.** Greenland looks bigger than Africa. For thematic comparison, use an equal-area projection.
7. **Privacy.** Even k-anonymized data with location can re-identify individuals at home/work hours. Coarsen or jitter.
8. **Date line / pole crossings.** Polygons that cross 180° longitude break naively.

## 5. Operations you'll use

| Operation | Function (geopandas) |
|---|---|
| Reproject | `gdf.to_crs(epsg=32633)` |
| Spatial join (which polygon contains each point) | `gpd.sjoin(points, polys, predicate='within')` |
| Buffer (e.g., 1 km around each point) | `gdf.geometry.buffer(1000)` (in projected CRS!) |
| Distance between geometries | `gdf1.distance(gdf2)` (in projected CRS) |
| Aggregate by region | `sjoin` + `groupby` |
| Simplify geometries | `gdf.geometry.simplify(tolerance)` |

## 6. Cleaning checklist

See [`data_cleaning/geospatial_cleaning.md`](../data_cleaning/geospatial_cleaning.md).

- [ ] Confirm CRS is set and correct.
- [ ] Reproject to a metric CRS before distance / area work.
- [ ] Validate geometries; fix invalid ones.
- [ ] Drop or fix empty geometries.
- [ ] Remove duplicate features.
- [ ] Snap near-duplicate vertices for clean topology.
- [ ] Audit address-to-coordinate (geocoding) quality — many "0, 0" results are failed geocodes.

## 7. Visualizations

See [`data_visualization/geospatial_viz.md`](../data_visualization/geospatial_viz.md).

| Map type | When |
|---|---|
| Choropleth | One value per region (equal-area projection!) |
| Graduated symbols | Magnitude at points |
| Heatmap / kernel density | Density of points |
| Hexbin / H3 | Aggregating points to grid cells, less biased than choropleth |
| Dot density | Counts (one dot per N items) |
| Flow / arrow map | Movement between locations |
| 3-D extruded | Height encodes a third variable |

## 8. References

- Lovelace, Nowosad, Muenchow. *Geocomputation with R* (concepts apply to Python).
- Bivand, Pebesma, Gómez-Rubio. *Applied Spatial Data Analysis*.
- geopandas, shapely, rasterio, pyproj documentation.
