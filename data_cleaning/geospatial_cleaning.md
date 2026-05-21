# Geospatial Cleaning

> **TL;DR** — Most geospatial bugs come from one of three places: the **CRS** wasn't set (or was wrong), the **geometries** are invalid, or the **lat/lon** were swapped. Fix those three and the rest is straightforward.

## 1. Confirm and set the CRS

```python
import geopandas as gpd
gdf = gpd.read_file('input.gpkg')
gdf.crs            # is it what you expected?
gdf.set_crs(epsg=4326, inplace=True, allow_override=False)
```

- If you didn't set a CRS, *every library* will assume something — and they assume different things.
- Most online data is EPSG:4326 (WGS 84, lat/lon).
- Web map tiles are EPSG:3857 (Web Mercator).
- Use `gdf.to_crs(epsg=...)` to reproject for distance / area work (a metric CRS like UTM).

## 2. Detect lat/lon swap

A common bug: someone stored coordinates as `(lat, lon)` but the consumer expects `(lon, lat)` (GeoJSON convention).

Sanity checks:
- For data inside a known country, plot the points; the wrong ones will sit on the wrong continent.
- If `lon > 90` or `lat > 180`, they're definitely swapped.

## 3. Validate geometries

```python
invalid = gdf[~gdf.is_valid]
gdf.geometry = gdf.geometry.make_valid()       # shapely 2.x
# or, for older shapely:
gdf.geometry = gdf.geometry.buffer(0)
```

`make_valid` handles self-intersections, slivers, and overlapping polygons. After fixing, run `is_valid` again to confirm.

## 4. Drop empty / null geometries

```python
gdf = gdf[~gdf.geometry.is_empty & gdf.geometry.notna()]
```

These cause crashes in spatial joins and silent zeros in area calculations.

## 5. Distance and area must be in a projected CRS

```python
gdf_m = gdf.to_crs(epsg=32633)   # UTM zone 33N for Central Europe
gdf_m['area_m2'] = gdf_m.geometry.area
gdf_m['perim_m'] = gdf_m.geometry.length
```

Computing area in EPSG:4326 returns square *degrees* — meaningless.

Pick a UTM zone based on the data's longitude, or use an equal-area CRS like Albers for area-comparison work.

## 6. Snap and simplify

Datasets often have:
- Vertices that are millimeters apart but should be identical (causes failed unions).
- Excessive vertex density on polygons that don't need it (slow rendering).

```python
gdf.geometry = gdf.geometry.simplify(tolerance=0.0001, preserve_topology=True)
```

For shared boundaries (e.g., country borders that should match between two datasets), use topological simplification (`topojson`) so they stay aligned.

## 7. Geocoding QA

Address → coordinates is noisy. Audit:

- Reject results with `confidence < threshold` from the geocoder.
- Reject results at suspicious "default" coordinates like `(0, 0)` or country centroids.
- Spot-check a sample against known addresses.

## 8. Spatial joins — common gotchas

```python
joined = gpd.sjoin(points, polygons, predicate='within')
```

- Predicate matters: `within`, `intersects`, `contains`, `touches`.
- Points exactly on a polygon boundary: behavior is library/version-dependent. If important, add a small inward buffer.
- One-to-many joins: a point in two overlapping polygons gives two rows. Decide policy.
- Performance: `sjoin` uses an R-tree by default; verify with `gdf.sindex`.

## 9. Deduplicate

Exact-geometry duplicates: `gdf.drop_duplicates('geometry')` (after canonicalization).

Near-duplicate: use `geopandas.sjoin_nearest` with a small threshold.

## 10. Privacy

- Even with names removed, location data can re-identify individuals. Trips that start at "home at 11pm" and end at "work at 9am" are unique to a small number of people in any city.
- Aggregation strategies: H3 cells, dropping the last decimal places of coordinates (1 decimal degree ≈ 11 km), or applying differential privacy.
- For published datasets, follow domain-specific rules (HIPAA for health, GDPR in the EU).

## 11. Checklist

- [ ] CRS is set and verified.
- [ ] Lat/lon order is correct.
- [ ] Invalid geometries fixed.
- [ ] Empty / null geometries removed.
- [ ] Distance / area work done in a projected CRS.
- [ ] Geocoded points audited.
- [ ] Spatial-join predicates chosen deliberately.
- [ ] Privacy controls applied before sharing.
