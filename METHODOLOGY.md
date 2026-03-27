# Methodology — Jawalgaon Dam Break PAR Analysis

> **Document type:** Technical methodology reference  
> **Scope:** Covers all 22 analytical cells of the Jupyter notebook pipeline  
> **Audience:** Researchers, dam safety engineers, emergency planners, GIS analysts  

---

## Table of Contents

1. [Conceptual Framework](#1-conceptual-framework)
2. [Data Ingestion & Validation](#2-data-ingestion--validation)
3. [Coordinate Reference System Management](#3-coordinate-reference-system-management)
4. [Geometry Pre-processing](#4-geometry-pre-processing)
5. [Hydraulic Raster Processing](#5-hydraulic-raster-processing)
6. [Inundation Polygon Generation](#6-inundation-polygon-generation)
7. [Arrival Time Reclassification](#7-arrival-time-reclassification)
8. [Hazard Classification](#8-hazard-classification)
9. [OpenStreetMap Infrastructure Query](#9-openstreetmap-infrastructure-query)
10. [Zonal Statistics](#10-zonal-statistics)
11. [PAR Calculation](#11-par-calculation)
12. [Evacuation Priority & Vulnerability Index](#12-evacuation-priority--vulnerability-index)
13. [Shelter Assignment](#13-shelter-assignment)
14. [Road & Infrastructure Inundation Analysis](#14-road--infrastructure-inundation-analysis)
15. [EAP Annexure Generation](#15-eap-annexure-generation)
16. [Alert Zone Classification](#16-alert-zone-classification)
17. [Assumptions](#17-assumptions)
18. [Limitations](#18-limitations)
19. [References](#19-references)

---

## 1. Conceptual Framework

This analysis follows the **ANCOLD / ICOLD Population At Risk (PAR) framework**, adapted for Indian dam safety practice under the Central Water Commission (CWC) Emergency Action Plan (EAP) guidelines.

The core question is: *Given a dam breach at time T=0, which people are at risk, where are they, how severe is the hazard they face, and how quickly must they be evacuated?*

Two failure modes are modelled independently:

| Scenario | Physical Mechanism | Typical Warning Time |
|----------|-------------------|----------------------|
| **Piping** | Internal erosion through the embankment; rapid uncontrolled failure | Very short (minutes to hours) |
| **Overtopping** | Flood wave overtops the crest; progressive failure | Moderate (hours), depending on hydrograph |

Both scenarios use the same analytical pipeline. Differences appear only in the input rasters and the time offset used to convert simulation clock time to *hours after breach*.

---

## 2. Data Ingestion & Validation

### 2.1 Input File Manifest
Cell 3 performs a pre-flight check across all 13 required input files (4 rasters × 2 scenarios + 5 shapefiles). Missing files are reported with their expected paths before any computation begins. This prevents mid-run failures on long-running analyses.

### 2.2 Shapefile Loading
Shapefiles are loaded with `geopandas.read_file()`. Column names are user-configurable in Cell 2 to accommodate variations across data providers (Census of India, municipal GIS departments, etc.).

Cell 5 (column inspection) should be run **before** Cell 2 is configured, as it prints the actual column names present in each shapefile.

---

## 3. Coordinate Reference System Management

### 3.1 Auto UTM Detection
A single function computes the optimal UTM zone from the settlement layer's geographic centroid:

```
zone = floor((mean_longitude + 180) / 6) + 1
hemisphere_code = 6 if latitude ≥ 0 else 7
EPSG = 32600 + zone  (Northern hemisphere)
```

For Solapur (approximately 17.7°N, 75.9°E), this yields UTM Zone 43N (EPSG:32643).

### 3.2 Reprojection Protocol
All vector layers are reprojected to the auto-detected UTM CRS before any geometric operation. All rasters are reprojected using `rasterio.warp.reproject()` with bilinear resampling for continuous bands (depth, velocity, WSE) and nearest-neighbour for categorical bands (arrival time class). Reprojected rasters are written to disk in `_reproj/` to avoid repeated I/O.

---

## 4. Geometry Pre-processing

### 4.1 Chaikin Corner-cutting Smoothing
Digitised settlement polygons frequently exhibit jagged edges from low-resolution heads-up digitising or automated extraction. These artefacts create spurious area measurements at polygon boundaries.

**Chaikin's algorithm** is applied iteratively: each vertex pair is replaced by two new vertices at ¼ and ¾ of the segment length. After *n* iterations, the polygon converges toward the B-spline approximation of the original control polygon.

The algorithm slightly *expands* the smoothed polygon — a conservative decision that prevents underestimation of inundated area at settlement fringes.

### 4.2 Dam Location Detection
The dam centroid is extracted automatically from the reservoir (`Submurgen.shp`) boundary by computing its geometric centroid projected to the dam-facing edge. This point is used as the origin for all *distance-from-dam* calculations and the river chainage sort order.

---

## 5. Hydraulic Raster Processing

### 5.1 Input Bands
Four hydraulic output bands are used from each scenario:

| Band | Symbol | Unit | Usage |
|------|--------|------|-------|
| Maximum water depth | *d* | m | Inundation extent, hazard classification, PAR |
| Maximum flow velocity | *v* | m/s | Hazard classification (DV product) |
| Arrival time | *t* | hours (simulation clock) | Evacuation window, travel-time table |
| Water surface elevation | *WSE* | m (MSL) | Infrastructure risk, bridge freeboard |

### 5.2 NoData Handling
Rasterio's `nodata` value is read from each raster's metadata. All pixels equal to `nodata` are masked to `NaN` before processing. This prevents phantom inundation in areas where the hydraulic model produced no output.

---

## 6. Inundation Polygon Generation

Inundation extents are derived by vectorising all pixels where `depth > 0`:

1. A binary mask (`depth > 0`) is created in the raster's native CRS.
2. `rasterio.features.shapes()` converts contiguous groups of masked pixels into GeoJSON-format polygons.
3. All polygons are assembled into a GeoDataFrame and unioned with `shapely.ops.unary_union`.
4. Optional Gaussian smoothing (`scipy.ndimage.gaussian_filter`) can be applied to the raster before vectorisation to reduce pixelation artefacts in the polygon boundary.

The resulting single-polygon inundation extent is saved as `inundation_{scenario}.shp`.

---

## 7. Arrival Time Reclassification

The hydraulic model outputs arrival time as **simulation clock hours** — the time since the model simulation started (which may be many hours before the actual breach event).

To produce *actionable* evacuation windows, arrival times must be expressed as **hours after breach (T=0)**:

```
t_relative = t_simulation − t_breach_offset
```

For the **overtopping** scenario, `t_breach_offset` is user-specified (`OVT_OFFSET_HRS = 19.0` for Jawalgaon). For the **piping** scenario, `t_breach_offset` is auto-detected as the minimum non-zero, non-nodata value in the arrival raster (the first pixel to be inundated is assumed to be at the dam toe, representing T=0).

Any reclassified value < 0 is clamped to 0 (areas already inundated at breach initiation). Values above 9999 are treated as "not reached within simulation period."

---

## 8. Hazard Classification

### 8.1 DV Product
Flood hazard is classified using the **Depth × Velocity (DV) product** (m²/s), a widely adopted scalar proxy for flood destructive potential that captures both hydrostatic and hydrodynamic forces on structures and persons:

```
DV = depth (m) × velocity (m/s)
```

### 8.2 Class Boundaries

| Class | DV (m²/s) | Equivalent Risk Description |
|-------|-----------|----------------------------|
| H1 | 0.00–0.25 | Passable on foot; minor nuisance flooding |
| H2 | 0.25–0.50 | Difficult to wade; risk to vulnerable persons |
| H3 | 0.50–1.00 | Dangerous for most people; vehicle instability |
| H4 | 1.00–2.00 | Life-threatening; structural damage to light buildings |
| H5 | 2.00–4.00 | Catastrophic; unreinforced masonry failure |
| H6 | >4.00 | Total destruction expected |

These thresholds follow the FEMA (2013) and ANCOLD (2012) frameworks, commonly applied in Indian dam safety practice under NDMA guidelines.

### 8.3 Implementation
The DV raster is computed pixel-wise from reprojected depth and velocity rasters using `numpy` broadcasting. Classification is applied with `numpy.digitize()` against the `HAZARD_CFG` threshold array. The result is written as a single-band, uint8 GeoTIFF.

---

## 9. OpenStreetMap Infrastructure Query

### 9.1 Overpass API Strategy
Infrastructure data is retrieved from the OpenStreetMap Overpass API using a **single bulk query** bounded by the inundation extent bounding box. A single request is used deliberately to avoid HTTP 429 (rate-limit) errors that arise from issuing many small queries.

The query fetches:
- All road segments (all `highway=*` tags)
- All points of interest (buildings, amenities, emergency facilities, religious sites)
- Water infrastructure (wells, canals, bridges, barrages)

### 9.2 POI Categorisation
OSM features are grouped into critical-infrastructure categories for emergency prioritisation:

| Category | OSM Tags | EAP Relevance |
|----------|----------|---------------|
| Medical | `amenity=hospital,clinic,pharmacy` | Evacuation of non-ambulatory patients |
| Education | `amenity=school,college` | Evacuation centres, vulnerable populations |
| Water Supply | `man_made=water_well`, `amenity=water_point` | Contamination risk post-flood |
| Religious | `amenity=place_of_worship` | Community assembly points |
| Emergency | `emergency=*` | Pre-positioned response resources |

---

## 10. Zonal Statistics

Per-settlement flood statistics are computed using `rasterstats.zonal_stats()`, which aggregates raster pixel values within each settlement polygon:

| Column Prefix | Statistic | Example |
|---------------|-----------|---------|
| `{sc}_dep_max` | Maximum depth (m) | `pip_dep_max` |
| `{sc}_dep_avg` | Mean depth (m) | `ovt_dep_avg` |
| `{sc}_vel_max` | Maximum velocity (m/s) | `pip_vel_max` |
| `{sc}_vel_avg` | Mean velocity (m/s) | `ovt_vel_avg` |
| `{sc}_wse_max` | Maximum WSE (m) | `pip_wse_max` |
| `{sc}_wse_avg` | Mean WSE (m) | `ovt_wse_avg` |
| `{sc}_arr_min` | Minimum arrival time, hours | `pip_arr_min` |
| `{sc}_arr_avg` | Mean arrival time, hours | `ovt_arr_avg` |

Where `sc` is `pip` (piping) or `ovt` (overtopping). All statistics are computed in the projected UTM CRS.

---

## 11. PAR Calculation

### 11.1 Proportional Area Method
PAR is estimated using the **proportional area intersection method**, which is the standard approach when sub-settlement population distribution data are unavailable:

```
Inundated_Fraction = Intersection_Area(settlement, inundation) / Total_Area(settlement)
PAR = Total_Population × Inundated_Fraction
```

Where population data allows, PAR is disaggregated by sex (`PAR_m`, `PAR_f`) using the same inundation fraction applied to male and female population counts.

### 11.2 Flood Status Labels
Each settlement is assigned a qualitative flood status based on the inundated fraction and spatial analysis:

| Condition | Label |
|-----------|-------|
| Surrounded on all sides | VILLAGE SURROUNDED BY FLOOD |
| All road access flooded | VILLAGE ISOLATED — ROAD ACCESS CUT |
| ≥ 95% area inundated | VILLAGE COMPLETELY FLOODED |
| 70–95% area inundated | VILLAGE HEAVILY FLOODED |
| 30–70% area inundated | VILLAGE MODERATELY FLOODED |
| 5–30% area inundated | VILLAGE PARTIALLY FLOODED |
| 0–5% area inundated | VILLAGE MARGINALLY AFFECTED |
| 0% area inundated | NOT AFFECTED |

Road isolation is tested by checking whether the nearest road segment outside the settlement boundary falls within the inundation polygon.

---

## 12. Evacuation Priority & Vulnerability Index

### 12.1 Evacuation Priority (P1–P5)
Priority codes are assigned based on the minimum flood arrival time at each settlement (i.e., when the first flood wave reaches the settlement boundary):

| Code | Time Window | Action Label | Rationale |
|------|-------------|--------------|-----------|
| P1 | 0–2 h | IMMEDIATE | Insufficient time for organised evacuation; emergency alert + self-evacuation |
| P2 | 2–4 h | URGENT | Narrow window; pre-positioning of buses/boats required |
| P3 | 4–6 h | HIGH PRIORITY | Managed evacuation feasible with pre-plan activation |
| P4 | 6–8 h | STANDARD | Full EAP-protocol evacuation |
| P5 | >8 h | PLANNED | Phased, non-emergency relocation possible |

### 12.2 Composite Vulnerability Index
A normalised composite index (0–1 scale) is computed to rank settlements beyond the binary priority code:

```
VI = w₁·(hazard_class/6) + w₂·(inund_pct/100) + w₃·(PAR/max_PAR) + w₄·(1 − dist/max_dist)
```

Default weights: w₁=0.30, w₂=0.25, w₃=0.25, w₄=0.20. Each component is min-max normalised across the settlement dataset. Higher VI = greater relative vulnerability.

---

## 13. Shelter Assignment

Each settlement is assigned its **nearest designated evacuation shelter** using Euclidean distance in the projected UTM CRS. If no shelter layer is provided, the column is populated with "No shelter data."

Shelter capacity (`SHELTER_CAP_COL`) is optionally reported alongside the assigned shelter name and distance (km). Overcapacity situations (PAR > shelter capacity) are flagged for manual review.

---

## 14. Road & Infrastructure Inundation Analysis

### 14.1 Road Segments
OSM road lines are spatially intersected with the inundation polygon. Each intersected segment is attributed with:
- Road type (`highway` tag class)
- Flood arrival time at midpoint (sampled from arrival-time raster)
- Maximum depth along segment
- Hazard class

A road-type summary table (`Road_Type_Summary.csv`) aggregates total inundated kilometres per road class.

### 14.2 Critical Infrastructure
Each OSM POI is point-sampled against the depth and arrival-time rasters to determine:
- Whether it falls within the inundation zone
- The flood arrival time (hrs after breach)
- The maximum depth at the feature location
- The hazard class (H1–H6)

All at-risk features are exported to `Critical_Infrastructure_At_Risk.csv` with coordinates for emergency mapping.

---

## 15. EAP Annexure Generation

### Annexure 3A
Standard CWC EAP format for settlement-level flood information. Columns per the CWC template: village name, distance from dam, flood arrival time, maximum depth, maximum velocity, PAR, evacuation priority, nearest shelter.

### Annexure 3B
PAR summary table, disaggregated by flood status category. Provides district-level totals suitable for inclusion in the EAP executive summary.

---

## 16. Alert Zone Classification

Settlements are classified into three standard EAP alert zones:

| Zone | Criterion | Emergency Response Level |
|------|-----------|--------------------------|
| Zone I (Immediate) | Arrival time < 2 hours | EAP Level 3 — Emergency |
| Zone II (Secondary) | Arrival time 2–6 hours | EAP Level 2 — Alert |
| Zone III (Warning) | Arrival time > 6 hours | EAP Level 1 — Warning |

These zones correspond to the siren/alert notification tiers in a standard dam EAP.

---

## 17. Assumptions

1. **Hydraulic model validity.** Raster inputs are assumed to be the best available hydraulic simulation outputs. No correction is applied for model uncertainty.
2. **Population uniformity.** Population is assumed uniformly distributed within settlement polygons. In reality, residential cores may be denser than the polygon-average.
3. **Census population currency.** Population figures reflect the most recent available census or survey data and do not account for seasonal migration or day/night variability.
4. **Instantaneous inundation.** PAR uses the maximum inundation extent, not a time-stepped analysis. It therefore represents the worst-case spatial extent.
5. **Road accessibility.** Road passability is based on geometric intersection only. Actual road conditions (embankment height, drainage) are not modelled.
6. **OSM completeness.** Infrastructure data quality depends on OSM contributor coverage, which is lower in rural Maharashtra than in urban areas.
7. **Shelter capacity.** Where capacity data are absent, shelters are assumed to have unlimited capacity for assignment purposes.
8. **Single DEM source.** The Sentinel 30 m DEM is used for hydraulic routing. Vertical errors in this DEM propagate into depth and velocity estimates.

---

## 18. Limitations

- This tool does **not** perform hydraulic modelling. It consumes outputs from an external model (HEC-RAS or equivalent).
- The **proportional area PAR method** is a conservative first approximation. For formal EAP submissions, field-validated population data at sub-settlement scale is recommended where available.
- OSM infrastructure data **may be incomplete or outdated** for rural areas. Field verification is required before operational use.
- The **Chaikin smoothing algorithm** modifies polygon geometry. Users should compare smoothed and original extents for their specific data.
- Alert zone boundaries are based on **travel time alone** and do not incorporate topographic impedance or evacuation route condition.

---

## 19. References

1. Central Water Commission (2018). *Guidelines for Preparation of Emergency Action Plan for Large Dams.* Government of India.
2. NDMA (2019). *National Guidelines on Dam Safety.* National Disaster Management Authority, India.
3. ANCOLD (2012). *Guidelines on Consequence Categories for Dams.* Australian National Committee on Large Dams.
4. FEMA (2013). *Selecting and Accommodating Inundation Maps in Emergency Action Plans.* FEMA P-64.
5. ICOLD (2017). *Dam Safety Management: Operational Phase of the Dam Life Cycle.* Bulletin 154.
6. Brunner, G.W. (2016). *HEC-RAS River Analysis System: 2D Modelling User's Manual.* US Army Corps of Engineers.
7. Jordahl, K. et al. (2020). *GeoPandas: Python tools for geographic data.* Zenodo.
8. Gillies, S. (2013). *Rasterio: geospatial raster I/O for Python programmers.* Mapbox.
