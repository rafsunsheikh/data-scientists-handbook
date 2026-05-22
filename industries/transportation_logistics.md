# Transportation and Logistics

> **TL;RT** — Transportation and logistics data centers on GPS traces, shipment events, and capacity records that span road, rail, sea, and air. The distinguishing challenges are map-matching noisy GPS to road networks, segmenting raw traces into trips and stops, reconciling scheduled vs. actual times (ETA vs. ETD), and the multi-modal nature of freight (truck → rail → ship → truck). Route optimization and ETA prediction are the canonical problems, but dwell-time analysis, load factor optimization, and network design are equally important for profitability. The data volume is moderate (thousands to millions of GPS points per day) but the temporal and spatial alignment is complex.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **GPS traces** | Vehicle location (lat, lon, alt, speed, heading) at 1–10 Hz |
| **Shipment events** | Pickup, departure, arrival, customs clearance, delivery |
| **Fleet telematics** | Engine RPM, fuel level, idle time, harsh braking, seatbelt |
| **GTFS / GTFS-realtime** | Public transit schedules, real-time vehicle positions, trip updates |
| **AIS (Automatic Identification System)** | Maritime vessel position, speed, course, vessel identity |
| **ADS-B (Automatic Dependent Surveillance-Broadcast)** | Aviation aircraft position, altitude, speed, callsign |
| **Capacity / load** | Truck capacity (weight, volume), container size, load factor |
| **On-time performance** | Scheduled vs. actual departure/arrival |
| **Fuel / emissions** | Fuel consumption, CO2, NOx, particulate |
| **Geospatial** | Road network, ports, airports, warehouses, delivery zones |

## 2. Common sources

| System | What |
|---|---|
| **Telematics providers** | Geotab, Samsara, Verizon Connect, SOTI, Motive |
| **TMS (Transportation Management System)** | Oracle TMS, SAP TM, Mercury Gate, Red Prism |
| **WMS (Warehouse Management System)** | Manhattan, Blue Yonder, SAP WM |
| **AIS aggregators** | MarineTraffic, FleetMon, VesselFinder |
| **ADS-B aggregators** | FlightAware, Flightradar24, ADS-B Exchange |
| **GTFS feeds** | Local transit agencies (bus, rail, subway) |
| **EDI (Electronic Data Interchange)** | 856 (ASN), 214 (shipment status), 204 (shipment tender) |
| **ELD (Electronic Logging Device)** | Hours of Service (HOS) for commercial drivers |
| **GPS hardware** | Trimble, Garmin, Teltonika, Queclink |
| **Route optimization** | Routific, OptimoRoute, Onfleet, Route4Me |

## 3. Standard schemas and formats

### GPS trace (typical)
```
telematics_id, timestamp (UTC),
latitude, longitude, altitude_m,
speed_kmh, heading_deg,
satellite_count, hdop,
ignition_status, event_code (hard_brake, harsh_turn, idle_start, idle_end)
```

### GTFS (static)
```
routes.txt: route_id, route_short_name, route_long_name, route_type (0=tram, 3=bus, 4=subway, 7=rail)
stops.txt: stop_id, stop_lat, stop_lon, stop_name
stop_times.txt: route_id, trip_id, stop_id, arrival_time, departure_time, stop_sequence
trips.txt: route_id, trip_id, service_id
calendar.txt / calendar_dates.txt: service dates
fare_attributes.txt, fare_rules.txt, transfers.txt, frequencies.txt
```

### AIS message (NMEA)
```
$--AIVDM,<type>,<number_of_messages>,<msg_seq>,<mmsi>,<payload>,<bit_count>,<check_sum>

Type 1/2/3: Position report (lat, lon, speed, course, heading)
Type 5: Static data (vessel name, dimensions, call sign, destination)
Type 18: Standard class B position report
Type 19: Extended class B position report
```

### EDI 214 (Shipment Status)
```
ST~214~0001
REF*2I*SHIPMENT_ID
REF*4F*EVENT_CODE (01=dispatched, 04=in_transit, 06=delivered, 08=exception)
DTM*307*20240115*802*1430  # departure date/time
DTM*214*20240115*802*1645  # arrival date/time
N9*DEST*WAREHOUSE_ID
```

## 4. Cleaning particulars

### 4.1 Map matching

GPS points rarely fall on the road. Map matching snaps GPS to the road network.

**Methods:**

- **Simple nearest:** Find nearest road segment. Fails on parallel roads, parking lots.
- **Hidden Markov Model (HMM):** Probabilistic — considers road connectivity and transition probabilities. Standard approach (e.g., OSRM, Valhalla, GraphHopper).
- **Kalman filter + map matching:** Combines GPS noise model with road geometry.

**Python:**

```python
import valhalla  # or osrm, or graphhopper

# Map match GPS trace to road network
result = valhalla.map_match(
    locations=[{"lat": 37.7749, "lon": -122.4194}, ...],
    shape_match="map_match_simple"  # or "flow_in_graph", "edge_snap"
)
```

### 4.2 Trip and stop segmentation

Raw GPS is a continuous trace. Segment into trips and stops.

**Stop detection:**

- **Speed threshold:** Vehicle speed < 5 km/h for > 60 seconds → stop.
- **Spatial clustering:** GPS points within 50m radius for > 5 minutes → stop location.
- **Ignition status:** Ignition off → stop; ignition on → trip start.

**Trip segmentation:**

- Split at stops.
- Handle gaps (GPS dropout > 5 minutes → new trip).
- Filter out non-driving (parking lot maneuvers, idling at lights).

### 4.3 ETA vs. actual times

| Field | Description |
|---|---|
| **ETD** | Estimated time of departure (planned or predicted) |
| **ATD** | Actual time of departure |
| **ETA** | Estimated time of arrival |
| **ATA** | Actual time of arrival |
| **Scheduled** | Planned times (from schedule or TMS) |

**Cleaning:**

- Distinguish planned vs. predicted ETAs (TMS plan vs. real-time prediction).
- Handle timezone issues (all times should be UTC).
- Flag impossible sequences (ATA < ATD, ATD < ETD by > 2 hours).

### 4.4 AIS and vessel data

- **MMSI validation:** 9-digit Maritime Mobile Service Identity. Check checksum.
- **Vessel type classification:** IMO classification code or AIS vessel type codes.
- **Dead reckoning:** During GPS outages, vessels move at constant speed/direction.
- **Port geofencing:** Define polygon around port to detect arrival/departure.

### 4.5 GTFS cleaning

- **Temporal consistency:** Arrival time must be after departure time for each stop.
- **Service calendar:** Check for overlapping or conflicting service dates.
- **Stop time headway:** Consecutive stops should have reasonable time gaps (not negative).
- **Shape accuracy:** GPS trace of vehicle should follow the GTFS shape.

### 4.6 Fuel and emissions

- **Fuel consumption rate:** Liters/hour or gallons/hour. Normalize by engine hours, not clock time.
- **Idle fuel:** Engine running but vehicle stationary — significant waste.
- **Emissions factors:** EPA or EU standards (Euro 6, Tier 4). CO2 per ton-km.

## 5. Standard analyses

### 5.1 Route optimization

| Analysis | Methods |
|---|---|
| **Vehicle Routing Problem (VRP)** | Clarke-Wright savings, LKH heuristic, OR-Tools CP-SAT |
| **Traveling Salesman Problem (TSP)** | Held-Karp, 2-opt, 3-opt, LKH |
| **Pickup and Delivery (PDVRP)** | Time windows, capacity constraints, backhauls |
| **Dynamic routing** | Real-time rerouting (traffic, weather, new orders) |
| **Last-mile optimization** | Cluster + route, crowdshipping, locker networks |
| **Multi-modal routing** | Truck + rail + ship combinations |

### 5.2 ETA prediction

| Analysis | Methods |
|---|---|
| **ETA regression** | GBM (XGBoost, LightGBM) with traffic, weather, historical features |
| **ETA distribution** | Quantile regression, survival analysis |
| **Route-based ETA** | Graph algorithms (Dijkstra, A*) + historical travel times |
| **Real-time ETA** | Kalman filter, particle filter with live traffic |
| **Arrival probability** | Probabilistic ETA (P(on-time) > 95%) |

### 5.3 Fleet analytics

| Analysis | Methods |
|---|---|
| **Driver behavior scoring** | Harsh braking, acceleration, cornering events per 100 km |
| **Fuel efficiency** | MPG/L per 100 km, idle time %, route-level comparison |
| **Maintenance scheduling** | Odometer-based, engine-hour-based, predictive (vibration, temperature) |
| **Utilization** | Active hours / total hours, load factor, empty miles % |
| **HOS compliance** | Hours of Service (FMCSA): 11 driving / 14 window / 30-min break |

### 5.4 Public transit

| Analysis | Methods |
|---|---|
| **Schedule adherence** | Actual vs. scheduled departure at each stop |
| **Headway regularity** | Standard deviation of headways on each route |
| **Load factor estimation** | Boarding/alighting counts from APTEC or smart card data |
| **Route performance** | Ridership per km, revenue per passenger, on-time rate |
| **Accessibility** | Isochrone maps (reachable destinations in 30 min) |

### 5.5 Maritime and aviation

| Analysis | Methods |
|---|---|
| **Port dwell time** | Time from arrival to departure at port |
| **Vessel utilization** | Days in port vs. days at sea |
| **Flight delay analysis** | Delay decomposition (airline, weather, ATC, airport) |
| **Slot optimization** | Airport slot allocation, turnaround time |
| **Route profitability** | Revenue per route, load factor, fuel cost |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Vehicle location | **Map** with GPS trace + current position |
| Route comparison | **Map** with multiple route overlays |
| On-time performance | **Heatmap** (route × day) |
| ETA accuracy | **Scatter** (predicted vs. actual) + Bland-Altman |
| Dwell time | **Box plot** by port/warehouse |
| Load factor over time | **Line chart** with capacity threshold |
| Transit coverage | **Isochrone map** (reachable in 15/30/45 min) |
| OD matrix | **Flow map** (origin → destination arrows, width = volume) |
| Driver events | **Timeline** (braking, acceleration, idle) |
| Fuel consumption | **Bar chart** by route or vehicle |

## 7. Regulation and ethics

| Regulation / standard | Scope |
|---|---|
| **FMCSA HOS** (US) | Hours of Service for commercial drivers |
| **GDPR** | Driver location data, ELD data |
| **EU ELD** | Electronic recording of driving time |
| **IMO SOLAS** | Maritime safety — AIS mandatory for vessels |
| **FAA regulations** (US) | Aviation operations, ADS-B In/Out |
| **IATA standards** | Air cargo, shipment tracking |
| **EDI X12 / EDIFACT** | Electronic data interchange standards |
| **DOT / ADR** | Hazardous materials transport |
| **Customs / sanctions** | OFAC screening for international freight |
| **Carbon reporting** | EU ETS, CORSIA (aviation), IMO 2020 (shipping) |

## 8. Public datasets

| Dataset | What |
|---|---|
| **NYC Taxi & Limousine Commission** | Yellow/taxi trip records (100M+ trips) |
| **Chennai MRTS / Singapore EZ-Link** | Public transit smart card data |
| **OpenStreetMap** | Road network, transit routes |
| **GTFS feeds** | Transit schedules for 1,000+ agencies |
| **MarineCadastre AIS** | US coastal vessel tracking |
| **FlightAware / ADS-B Exchange** | Flight data (some public) |
| **FRA On-Time Performance** | US railroad on-time data |
| **BTS (Bureau of Transportation)** | US aviation, rail, transit statistics |
| **Port of LA/Long Beach** | Container throughput, vessel arrivals |
| **Uber Movement** | Travel time matrices (selected cities) |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `geopandas`, `contextily` | Geospatial analysis and mapping |
| `osmnx` | OpenStreetMap network extraction |
| `valhalla`, `osrm`, `graphhopper` | Map matching, routing, isochrones |
| `h3` | Hexagonal indexing for spatial aggregation |
| `ortools` | Route optimization (CP-SAT solver) |
| `pandas`, `polars` | GPS trace processing |
| `fiona`, `shapely` | Geospatial geometry operations |
| `prophet`, `statsmodels` | ETA and demand forecasting |
| `scikit-learn`, `xgboost` | ETA prediction, classification |
| `aislib` (R) / `pymaplass` | AIS data processing |
| `gtfs-kit` | GTFS validation and analysis |

## 10. References

- Toth, P. & Vanderbei, R. *Linear Programming: Foundations and Extensions*. (VRP/TSP)
- Dantzig, G. & Ramser, J. *The Truck Dispatching Problem*. Management Science (1959).
- Clarke, G. & Wright, J. *Scheduling of vehicles from a central depot*. Operations Research (1964).
- OSRM (Open Source Routing Machine) Documentation — https://project-osrm.org/
- Valhalla Documentation — https://github.com/valhalla/valhalla
- IMO AIS Manual — https://www.imo.org/
- GTFS Specification — https://gtfs.org/
