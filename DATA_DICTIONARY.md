# Data Dictionary — Jawalgaon Dam Break PAR Analysis

> Complete reference for all input fields, output columns, raster bands, and file formats produced by this analysis.

---

## Input Shapefiles

### `village_polygon.shp`
Settlement boundary polygons with population attributes.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | String | ✅ Yes | Village or settlement name *(configurable via `NAME_COLUMN`)* |
| `Pop` | Integer | ✅ Yes | Total resident population *(configurable via `POP_COLUMN`)* |
| `Pop_M` | Integer | ⬜ Optional | Male population *(configurable via `POP_MALE_COLUMN`)* |
| `Pop_F` | Integer | ⬜ Optional | Female population *(configurable via `POP_FEMALE_COLUMN`)* |
| `HH` | Integer | ⬜ Optional | Number of households *(configurable via `HH_COLUMN`)* |
| `Shelter` | String | ⬜ Optional | Designated evacuation shelter name *(configurable via `SHELTER_NAME_COL`)* |
| `geometry` | Polygon | ✅ Yes | Settlement boundary polygon |

### `shelter locations_*.shp`
Designated evacuation shelter points.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Shelter` | String | ✅ Yes | Shelter facility name *(configurable via `SHELTER_NAME_COL`)* |
| `Capacity` | Integer | ⬜ Optional | Maximum persons accommodated *(configurable via `SHELTER_CAP_COL`)* |
| `geometry` | Point/Polygon | ✅ Yes | Shelter location |

---

## Input Rasters

All rasters must be GeoTIFF format. CRS can be any projected or geographic system — the notebook reprojects automatically.

| Variable | File Pattern | Unit | Band Count | Notes |
|----------|-------------|------|-----------|-------|
| Maximum water depth | `Depth (Max)…tif` | metres | 1 | Must be ≥ 0; NoData = dry cells |
| Maximum flow velocity | `Velocity (Max)…tif` | m/s | 1 | Must be ≥ 0 |
| Flood arrival time | `Arrival Time (Max)…tif` | hours (simulation clock) | 1 | Converted to T=0 relative in Cell 10 |
| Water surface elevation | `WSE (Max)…tif` | metres (MSL) | 1 | Used for infrastructure clearance analysis |

---

## Output CSVs

### `Annexure_3A_Piping.csv` / `Annexure_3A_Overtopping.csv`
CWC EAP standard settlement table.

| Column | Type | Description |
|--------|------|-------------|
| `Name` | String | Village name |
| `Population` | Integer | Total population |
| `Distance_km` | Float | Distance from dam along river (km) |
| `Arrival_hrs` | Float | Flood arrival time (hours after breach) |
| `Max_Depth_m` | Float | Maximum inundation depth (m) |
| `Max_Velocity_ms` | Float | Maximum flow velocity (m/s) |
| `Inund_Pct` | Float | Percentage of settlement area inundated |
| `PAR` | Integer | Population at risk |
| `Flood_Status` | String | Flood status label (e.g., HEAVILY FLOODED) |
| `Evac_Priority` | String | Evacuation priority code (P1–P5) |
| `Evac_Window` | String | Time window label (e.g., "0 to 2 hours") |
| `Evac_Label` | String | Action label (e.g., IMMEDIATE) |
| `Nearest_Shelter` | String | Assigned evacuation shelter name |
| `Shelter_Dist_km` | Float | Distance to assigned shelter (km) |
| `Alert_Zone` | String | Zone I / Zone II / Zone III |

### `Annexure_3B_Piping_PAR.csv` / `Annexure_3B_Overtopping_PAR.csv`
PAR summary by flood status category.

| Column | Type | Description |
|--------|------|-------------|
| `Flood_Status` | String | Status category |
| `Village_Count` | Integer | Number of settlements in this category |
| `Total_Population` | Integer | Sum of all residents |
| `PAR` | Integer | Sum of population at risk |
| `PAR_Pct` | Float | PAR as % of total population |

### `Zonal_Statistics.csv`
Raster statistics per settlement polygon.

| Column | Type | Description |
|--------|------|-------------|
| `Name` | String | Village name |
| `pip_dep_max` | Float | Maximum depth — piping (m) |
| `pip_dep_avg` | Float | Mean depth — piping (m) |
| `pip_vel_max` | Float | Maximum velocity — piping (m/s) |
| `pip_vel_avg` | Float | Mean velocity — piping (m/s) |
| `pip_wse_max` | Float | Maximum WSE — piping (m MSL) |
| `pip_wse_avg` | Float | Mean WSE — piping (m MSL) |
| `pip_arr_min` | Float | Earliest arrival — piping (hrs after breach) |
| `pip_arr_avg` | Float | Mean arrival — piping (hrs after breach) |
| `ovt_dep_max` | Float | Maximum depth — overtopping (m) |
| `ovt_dep_avg` | Float | Mean depth — overtopping (m) |
| `ovt_vel_max` | Float | Maximum velocity — overtopping (m/s) |
| `ovt_vel_avg` | Float | Mean velocity — overtopping (m/s) |
| `ovt_wse_max` | Float | Maximum WSE — overtopping (m MSL) |
| `ovt_wse_avg` | Float | Mean WSE — overtopping (m MSL) |
| `ovt_arr_min` | Float | Earliest arrival — overtopping (hrs after breach) |
| `ovt_arr_avg` | Float | Mean arrival — overtopping (hrs after breach) |

### `Master_Distance_From_Dam.csv`
All features (settlements, infrastructure, roads) sorted by distance from dam along river chainage.

| Column | Type | Description |
|--------|------|-------------|
| `Feature_Name` | String | Feature name or ID |
| `Feature_Type` | String | Type: Settlement / Road / POI / Bridge |
| `Distance_km` | Float | Cumulative river chainage from dam (km) |
| `Lat` | Float | WGS84 latitude |
| `Lon` | Float | WGS84 longitude |
| `pip_arr_hrs` | Float | Piping arrival time (hrs after breach) |
| `ovt_arr_hrs` | Float | Overtopping arrival time (hrs after breach) |
| `Alert_Zone` | String | Zone I / Zone II / Zone III |

### `Critical_Infrastructure_At_Risk.csv`
OSM point features within the inundation zone.

| Column | Type | Description |
|--------|------|-------------|
| `osm_id` | String | OpenStreetMap feature ID |
| `name` | String | Feature name (if available) |
| `category` | String | Infrastructure category (Medical, Education, etc.) |
| `amenity` | String | Raw OSM `amenity` tag value |
| `lat` | Float | WGS84 latitude |
| `lon` | Float | WGS84 longitude |
| `pip_arr_hrs` | Float | Piping arrival time (hrs after breach) |
| `pip_depth_m` | Float | Maximum depth at point — piping (m) |
| `pip_hazard` | String | Hazard class at point — piping (H1–H6) |
| `ovt_arr_hrs` | Float | Overtopping arrival time (hrs after breach) |
| `ovt_depth_m` | Float | Maximum depth at point — overtopping (m) |
| `ovt_hazard` | String | Hazard class at point — overtopping (H1–H6) |
| `inundated` | Boolean | True if within inundation polygon |

### `Roads_Inundation.csv`
Road segments within or adjacent to the inundation zone.

| Column | Type | Description |
|--------|------|-------------|
| `osm_id` | String | OSM way ID |
| `highway` | String | Road type (primary, secondary, track, etc.) |
| `name` | String | Road name (if available) |
| `length_km` | Float | Segment length (km) |
| `inundated_km` | Float | Length within inundation zone (km) |
| `arr_min_hrs` | Float | Earliest arrival at any point on segment (hrs) |
| `depth_max_m` | Float | Maximum depth along segment (m) |
| `hazard_class` | String | Maximum hazard class along segment |

### `Flood_Wave_Travel_Time.csv`
Progressive flood wave front arrival table.

| Column | Type | Description |
|--------|------|-------------|
| `Checkpoint` | String | Named location or km-distance marker |
| `Distance_km` | Float | Distance from dam (km) |
| `pip_arr_hrs` | Float | Piping scenario arrival (hrs after breach) |
| `ovt_arr_hrs` | Float | Overtopping scenario arrival (hrs after breach) |
| `pip_peak_depth` | Float | Peak depth at checkpoint — piping (m) |
| `ovt_peak_depth` | Float | Peak depth at checkpoint — overtopping (m) |

### `Road_Type_Summary.csv`
Aggregated road inundation by class.

| Column | Type | Description |
|--------|------|-------------|
| `highway` | String | Road type |
| `total_km` | Float | Total length of this road type in study area (km) |
| `pip_inundated_km` | Float | Length inundated — piping (km) |
| `ovt_inundated_km` | Float | Length inundated — overtopping (km) |
| `pip_inundated_pct` | Float | % of total length inundated — piping |
| `ovt_inundated_pct` | Float | % of total length inundated — overtopping |

### `OSM_POI_Category_Summary.csv`
Count of at-risk POIs by category.

| Column | Type | Description |
|--------|------|-------------|
| `Category` | String | Infrastructure category |
| `Total_Count` | Integer | Total OSM features in category |
| `pip_at_risk` | Integer | At risk — piping scenario |
| `ovt_at_risk` | Integer | At risk — overtopping scenario |

### `OSM_Water_Wells.csv`
Water supply infrastructure at risk.

| Column | Type | Description |
|--------|------|-------------|
| `osm_id` | String | OSM feature ID |
| `name` | String | Well name (if available) |
| `lat`, `lon` | Float | WGS84 coordinates |
| `pip_depth_m` | Float | Flood depth at well — piping (m) |
| `ovt_depth_m` | Float | Flood depth at well — overtopping (m) |
| `contamination_risk` | String | Risk level based on depth |

---

## Output Shapefiles

### `inundation_piping.shp` / `inundation_overtopping.shp`
Single-polygon inundation extent.

| Field | Type | Description |
|-------|------|-------------|
| `scenario` | String | `piping` or `overtopping` |
| `area_ha` | Float | Total inundated area (hectares) |
| `area_km2` | Float | Total inundated area (km²) |
| `geometry` | MultiPolygon | Inundation boundary |

### `settlements_piping.shp` / `settlements_overtopping.shp`
Settlements with all PAR attributes attached.

Contains all columns from `village_polygon.shp` plus all columns from `Annexure_3A_*.csv` and `Zonal_Statistics.csv` for the respective scenario.

Additional computed fields:

| Field | Type | Description |
|-------|------|-------------|
| `{sc}_orig_ha` | Float | Original settlement area (ha) |
| `{sc}_inund_ha` | Float | Inundated area (ha) |
| `{sc}_inund_pct` | Float | Inundated % |
| `{sc}_PAR` | Integer | Population at risk |
| `{sc}_PAR_m` | Integer | Male PAR |
| `{sc}_PAR_f` | Integer | Female PAR |
| `{sc}_PAR_hh` | Integer | Households at risk |
| `{sc}_flood_st` | String | Flood status label |
| `{sc}_surrounded` | Boolean | Settlement completely surrounded |
| `{sc}_isolated` | Boolean | All road access cut |
| `{sc}_ev_ord` | Integer | Evacuation order (1–5) |
| `{sc}_ev_win` | String | Time window label |
| `{sc}_ev_cod` | String | Priority code (P1–P5) |
| `{sc}_ev_lbl` | String | Action label |
| `{sc}_vuln_idx` | Float | Composite vulnerability index (0–1) |
| `sh_name` | String | Nearest shelter name |
| `sh_dist_km` | Float | Distance to nearest shelter (km) |
| `alert_zone` | String | Zone I / II / III |

Where `{sc}` = `pip` or `ovt`.

---

## Output Rasters

### `hazard_piping.tif` / `hazard_overtopping.tif`
H1–H6 classified hazard raster.

| Band | Data Type | Description |
|------|-----------|-------------|
| 1 | uint8 | Hazard class: 0=Dry, 1=H1, 2=H2, 3=H3, 4=H4, 5=H5, 6=H6 |

### `arrival_piping_relative.tif` / `arrival_overtopping_relative.tif`
Flood arrival time raster, hours after breach.

| Band | Data Type | Description |
|------|-----------|-------------|
| 1 | float32 | Hours after breach; NoData = not reached |

---

## Hazard Classification Reference

| Code | Integer Value | DV Range (m²/s) | Label | Human Hazard |
|------|--------------|-----------------|-------|-------------|
| H1 | 1 | 0.00–0.25 | Very Low | Can stand/wade |
| H2 | 2 | 0.25–0.50 | Low | Difficult to walk |
| H3 | 3 | 0.50–1.00 | Medium | Dangerous to walk |
| H4 | 4 | 1.00–2.00 | High | Life-threatening |
| H5 | 5 | 2.00–4.00 | Very High | Structural damage |
| H6 | 6 | >4.00 | Extreme | Total destruction |

---

## Evacuation Priority Reference

| Code | Order | Time Window | Action Label |
|------|-------|-------------|--------------|
| P1 | 1 | 0–2 hours | IMMEDIATE |
| P2 | 2 | 2–4 hours | URGENT |
| P3 | 3 | 4–6 hours | HIGH PRIORITY |
| P4 | 4 | 6–8 hours | STANDARD |
| P5 | 5 | 8+ hours | PLANNED |

---

## Flood Status Labels

| Label | Trigger Condition |
|-------|------------------|
| VILLAGE SURROUNDED BY FLOOD | Settlement geometry fully enclosed by inundation polygon |
| VILLAGE ISOLATED — ROAD ACCESS CUT | All road segments to/from settlement are inundated |
| VILLAGE COMPLETELY FLOODED | Inundated fraction ≥ 95% |
| VILLAGE HEAVILY FLOODED | Inundated fraction 70–95% |
| VILLAGE MODERATELY FLOODED | Inundated fraction 30–70% |
| VILLAGE PARTIALLY FLOODED | Inundated fraction 5–30% |
| VILLAGE MARGINALLY AFFECTED | Inundated fraction > 0% but < 5% |
| NOT AFFECTED | No intersection with inundation polygon |
