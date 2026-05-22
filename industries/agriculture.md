# Agriculture

> **TL;RT** — Agriculture data spans satellite and drone imagery, ground-level soil and weather sensors, yield monitors, and machinery telemetry, all tightly coupled to growing-season calendars and geographic boundaries. The distinguishing challenges are atmospheric correction of satellite imagery, cloud/shadow masking, aligning data layers at different spatial resolutions (satellite pixels vs. field boundaries vs. soil maps), and fitting analyses to growing-season windows that vary by region and year. Yield prediction is the canonical problem, but crop classification, disease detection, irrigation scheduling, and variable-rate prescription are increasingly important as precision agriculture scales.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Satellite imagery** | Landsat (30m), Sentinel-2 (10m), Planet (3m), MODIS (250m–1km) — multispectral (RGB + NIR + SWIR) |
| **Drone imagery** | RGB, multispectral (Red Edge, NIR), thermal — cm-level resolution |
| **Soil sensors** | Moisture (volumetric water content), EC (electrical conductivity), pH, NPK (nitrogen, phosphorus, potassium), temperature |
| **Weather** | Temperature, precipitation, humidity, wind, solar radiation, growing-degree days |
| **Yield monitor** | Combine harvester data: yield (bu/acre or t/ha), moisture, GPS position, speed |
| **Machinery telemetry** | John Deere Operations Center: planting depth, seed count, fertilizer rate, spray volume |
| **Geospatial** | Field boundaries (parcel), soil maps (SSURGO), topography (DEM), zoning |
| **Market prices** | Commodity futures (CBOT, ICE), cash prices by region, basis |
| **Pest / disease** | Scouting reports, trap counts, lab test results |
| **Irrigation** | Valve status, flow rate, soil moisture setpoints, evapotranspiration |

## 2. Common sources

| System | What |
|---|---|
| **Satellite providers** | ESA Copernicus (Sentinel), USGS (Landsat), Planet, Maxar, MODIS (NASA) |
| **USDA** | Cropland Data Layer (CDL), NASS surveys, WASDE reports, FSA records |
| **FAO** | Global agricultural statistics (FAOSTAT) |
| **Climate data** | NOAA, CHIRPS (rainfall), ERA5 (reanalysis), GFS (forecast) |
| **Precision ag platforms** | John Deere Operations Center, Climate FieldView, Granular, Farmers Edge |
| **Drone operators** | DJI, senseFly, Pix4D — custom flights |
| **Soil mapping** | SSURGO (US), ISRIC World Soil Information |
| **Weather stations** | AgWeatherNet, CoCoRaHS, USDA NRCS SNOTEL |
| **Market data** | CBOT, ICE, local grain elevators, cooperatives |

## 3. Standard schemas and formats

### Satellite imagery (GeoTIFF)
```
# Raster format — each pixel has values for multiple bands
file: sentinel2_band4.tif  # Band 4 = Red (10m resolution)
file: sentinel2_band8.tif  # Band 8 = NIR (10m resolution)

# Bands (Sentinel-2):
# B2: Blue (10m), B3: Green (10m), B4: Red (10m), B8: NIR (10m)
# B5, B6, B7: Red Edge (20m), B8a: NIR narrow (20m)
# B11, B12: SWIR (20m), B1: Coastal aerosol (60m)
```

### NDVI (Normalized Difference Vegetation Index)
```
NDVI = (NIR - Red) / (NIR + Red)
Range: -1 to +1
  -1 to 0: Water, clouds, bare soil
  0 to 0.3: Sparse vegetation
  0.3 to 0.6: Moderate vegetation
  0.6 to 1.0: Dense vegetation
```

### Yield monitor (typical)
```
timestamp, gps_lat, gps_lon,
yield_bu_per_acre, grain_moisture_pct,
forward_speed_mph, header_width_ft,
area_accumulated_acres, machine_id, crop_type
```

### Field boundary (GeoJSON)
```json
{
    "type": "Feature",
    "geometry": {
        "type": "Polygon",
        "coordinates": [[[-122.0, 37.0], [-122.0, 37.1], [-121.9, 37.1], [-121.9, 37.0], [-122.0, 37.0]]]
    },
    "properties": {
        "field_id": "FIELD_001",
        "crop": "corn",
        "variety": "P1132AM",
        "plant_date": "2024-04-15",
        "acres": 245.5,
        "soil_type": "San Joaquin loam"
    }
}
```

## 4. Cleaning particulars

### 4.1 Atmospheric correction

Raw satellite DN (Digital Number) values are not physically meaningful. Convert to surface reflectance.

**Steps:**

1. **DN to top-of-atmosphere (TOA) reflectance:** Apply radiometric calibration coefficients.
2. **TOA to surface reflectance:** Correct for atmospheric scattering (Rayleigh, aerosol) and absorption (water vapor, ozone).
3. **Tools:** `rasterio`, `sentinel2msi`, `STAC` clients, Google Earth Engine.

**Common tools:**

- **DOS (Dark Object Subtraction):** Simple, assumes dark pixels have zero reflectance.
- **6S model:** Second Simulation of Satellite Signal (more accurate).
- **Sen2Cor:** Built-in atmospheric correction for Sentinel-2.
- **LaSRC:** Landsat Surface Reflectance Code.

### 4.2 Cloud and shadow masking

Clouds and shadows corrupt imagery. Mask them out before analysis.

**Methods:**

- **QA bands:** Sentinel-2 has a QA60 band with cloud confidence scores.
- **NDVI threshold:** Clouds have very low NDVI (similar to water).
- **Brightness threshold:** Clouds are very bright in visible bands.
- **Scene classification (S2C):** ESA provides scene classification codes.

**Python:**

```python
import rasterio
import numpy as np

# Load Sentinel-2 bands
with rasterio.open("S2A_L2A_T32VSK_20240615_B04.jp2") as red_src:
    red = red_src.read(1).astype(float)
with rasterio.open("S2A_L2A_T32VSK_20240615_B08.jp2") as nir_src:
    nir = nir_src.read(1).astype(float)

# Calculate NDVI
ndvi = (nir - red) / (nir + red + 1e-10)

# Mask clouds (QA band — bit 10 = cloud, bit 11 = cirrus)
with rasterio.open("S2A_L2A_T32VSK_20240615_QA60.jp2") as qa_src:
    qa = qa_src.read(1)
    cloud_mask = (qa & (1 << 10)) == 0  # no cloud
    ndvi = np.where(cloud_mask, ndvi, np.nan)
```

### 4.3 Georeferencing alignment

Different data layers have different resolutions:

| Layer | Resolution |
|---|---|
| Sentinel-2 | 10m (red, NIR), 20m (Red Edge, SWIR), 60m (coastal aerosol) |
| Landsat | 30m (most), 100m (thermal) |
| Planet | 3m |
| Soil maps (SSURGO) | Varies (often 10–50m) |
| Field boundaries | Exact (survey-grade) |
| Weather stations | Point (interpolated to grid) |

**Alignment strategy:**

- Resample all layers to the finest common resolution (e.g., 10m).
- Use nearest-neighbor for categorical layers (soil type, land cover).
- Use bilinear or cubic for continuous layers (temperature, precipitation).

### 4.4 Growing-season windows

Agricultural analyses must be fit to the growing season:

- **Planting date:** Varies by crop, region, year.
- **Emergence:** ~7–14 days after planting (corn, soy).
- **Rapid growth:** Vegetative stage — NDVI increases rapidly.
- **Flowering / pollination:** Critical period — yield determined.
- **Physiological maturity:** Grain fill, drying.
- **Harvest:** Yield measured.

**Growing Degree Days (GDD):**

```
GDD = (T_max + T_max) / 2 - T_base
T_base varies by crop (corn: 10°C, wheat: 0°C)
Cumulative GDD from planting determines growth stage.
```

### 4.5 Yield monitor cleaning

- **Quality flag:** Yield monitors have accuracy flags (based on moisture, speed, header position).
- **Grid averaging:** Aggregate to 30×30 ft cells to reduce noise.
- **Moisture correction:** Convert to standard moisture (corn: 15.5%, soy: 13%).
- **Geospatial alignment:** Match yield cells to field boundaries and soil maps.

### 4.6 Sensor calibration

- **Soil sensors:** Drift over time. Calibrate against lab measurements.
- **Drone multispectral:** Use reflectance panels (white, gray, black) before each flight.
- **Yield monitors:** Calibrate at start of season (flow calibration, moisture calibration).

## 5. Standard analyses

### 5.1 Crop classification

| Analysis | Methods |
|---|---|
| **Crop type mapping** | Random Forest, SVM, U-Net on satellite time series |
| **Growth stage detection** | NDVI/EVI time series, derivative analysis |
| **Area estimation** | Stratified sampling, area-weighted classification |
| **Land cover classification** | Random Forest, deep learning (SegNet, U-Net) |

### 5.2 Yield prediction

| Analysis | Methods |
|---|---|
| **Yield forecasting** | GDD + NDVI + weather + soil features → regression, GBM |
| **Early-season prediction** | Satellite NDVI at key growth stages |
| **In-season adjustment** | Update predictions as new imagery arrives |
| **County/state aggregate** | Top-down aggregation from field-level predictions |

**Key features:**

- NDVI/EVI at key growth stages (V6, R1, R6 for corn).
- Growing degree days accumulated.
- Precipitation (deficit or excess at key stages).
- Soil type and drainage class.
- Planting date and variety.

### 5.3 Disease and pest detection

| Analysis | Methods |
|---|---|
| **Leaf disease** | CNN on drone imagery (corn leaf spot, wheat rust) |
| **Nutrient deficiency** | Spectral indices (N: NDVI, NDRSI; P: PSRI; K: KNDVI) |
| **Weed detection** | Object detection (YOLO, Faster R-CNN) on drone imagery |
| **Pest pressure** | Trap counts + weather models (degree-day models for pest lifecycle) |

### 5.4 Irrigation scheduling

| Analysis | Methods |
|---|---|
| **Soil moisture monitoring** | Sensor readings + ET (evapotranspiration) modeling |
| **ET estimation** | FAO-56 Penman-Monteith, remote sensing (SEBAL, METRIC) |
| **Deficit irrigation** | Optimize water use vs. yield loss |
| **Variable-rate irrigation** | Soil map + moisture sensor → zone-level control |

### 5.5 Variable-rate prescription

| Analysis | Methods |
|---|---|
| **VRT (Variable-Rate Technology)** | Yield map → prescription map (more seed where yield potential is high) |
| **Nutrient management** | Soil test + yield goal → NPK prescription |
| **Chemical application** | Weed map → spot-spray prescription |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Vegetation health over time | **NDVI time series** (line chart with growing season shading) |
| Field variability | **Choropleth** (yield map, NDVI map) |
| Soil moisture | **Heatmap** (depth × time) |
| Growing degree days | **Line chart** (cumulative GDD vs. threshold by growth stage) |
| Weather vs. yield | **Scatter** (precipitation × yield) with regression |
| Crop type distribution | **Pie / donut chart** by field or region |
| Irrigation events | **Bar chart** (daily ET vs. irrigation applied) |
| Pest pressure | **Map** with trap count as point size |
| Yield comparison | **Box plot** (variety × yield) |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **USDA / FSA records** | Farm program participation, CRP, conservation compliance |
| **Crop insurance** | Actual Production History (APH) — required for coverage |
| **Environmental compliance** | EPA (pesticide runoff), Clean Water Act, Clean Air Act |
| **Data privacy** | Farm data privacy acts (state-level in US); data ownership disputes |
| **GDPR** | EU farm data (farmers' personal data) |
| **Conservation programs** | CRP, CSP — reporting requirements |
| **Organic certification** | NOP (National Organic Program) record-keeping |
| **Water rights** | Irrigation water allocation, groundwater sustainability (SGMA in CA) |

### Data ownership

Farmers generate data from their equipment and land. Questions about ownership:

- **John Deere / precision ag platforms:** Terms of service may claim rights to farm data.
- **Farmers' rights:** State farm data privacy acts (IA, SD, ND, KS, etc.) restrict data sharing.
- **Cooperative data sharing:** Some farmers pool data for benchmarking (e.g., Yield Star).

## 8. Public datasets

| Dataset | What |
|---|---|
| **USDA Cropland Data Layer (CDL)** | Annual crop type map (US, 30m) |
| **Sentinel-2** | ESA multispectral imagery (10m, free) |
| **Landsat** | USGS multispectral imagery (30m, free) |
| **CHIRPS** | NOAA precipitation estimates (0.05°, free) |
| **ERA5** | ECMWF reanalysis weather (free) |
| **SSURGO** | USDA soil survey (free) |
| **FAOSTAT** | FAO global agricultural statistics |
| **USDA NASS** | Crop reports, surveys (free) |
| **USDA WASDE** | World Agricultural Supply and Demand Estimates |
| **Climate Engine** | Climate data portal (free) |
| **OpenDroneMap** | Drone processing (open-source) |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `rasterio` | Raster I/O, geospatial processing |
| `geopandas` | Vector geospatial analysis |
| `xarray` | Labeled multi-dimensional arrays (satellite time series) |
| `rioxarray` | Rasterio + xarray integration |
| `scikit-learn`, `xgboost` | Classification, regression |
| `pytorch`, `tensorflow` | Deep learning (U-Net, YOLO for imagery) |
| `opencv`, `albumentations` | Image processing, augmentation |
| `contextily`, `folium` | Map rendering |
| `pandas`, `polars` | Tabular data |
| `statsmodels` | Time series, regression |
| `QGIS` | Desktop GIS (open-source) |
| `Google Earth Engine` | Cloud-based satellite imagery analysis |
| `sentinel2msi` | Sentinel-2 data access and processing |

## 10. References

- Rouse, J. et al. *Monitoring the current year's corn crop with spectral reflectance.* USDA (1973). (NDVI introduction)
- FAO-56: *Crop Water Requirements* — Allen, R. et al. (1998).
- USDA NASS Cropland Data Layer Documentation — https://www.nass.usda.gov/Research_and_Science/Cropland/Release_and_Quality_Information/index.php
- ESA Sentinel-2 User Handbook — https://sentinel.esa.int/documents/247904/
- Friedl, M. et al. *MODIS Land Cover Type 13 Global Product.* Remote Sensing of Environment (2002).
