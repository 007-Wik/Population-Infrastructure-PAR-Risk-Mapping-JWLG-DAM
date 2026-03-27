# 🌊 Jawalgaon Dam Break — Population At Risk (PAR) Analysis

<div align="center">

![GIS](https://img.shields.io/badge/GIS-GeoPandas%20%7C%20Rasterio-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Scenarios](https://img.shields.io/badge/Scenarios-Piping%20%7C%20Overtopping-blue?style=for-the-badge)
![Region](https://img.shields.io/badge/Region-Solapur%2C%20Maharashtra%2C%20India-FF6B35?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

**Open-source geospatial framework for dam-break Population-at-Risk assessment**  
*Dual-scenario flood modelling · H1–H6 hazard zoning · Evacuation priority triage · EAP-compliant outputs*

[▶ Open in Colab](https://colab.research.google.com/github/YOUR_USERNAME/jawalgaon-dam-par-analysis/blob/main/notebooks/Jawalgaon_PAR_Complete.ipynb) · [📊 View Methodology](METHODOLOGY.md) · [📖 Data Dictionary](DATA_DICTIONARY.md) · [🤝 Contributing](CONTRIBUTING.md)

</div>

---

## Why This Repository Matters

Dam failures are among the most catastrophic natural–technical disasters, capable of displacing hundreds of thousands of people within hours of a breach. Yet in India, **standardised, reproducible, open-source tools** for dam-break Population-at-Risk (PAR) assessments remain scarce. EAP documentation is frequently produced in siloed, non-auditable workflows that cannot be readily validated or adapted.

This repository changes that for Jawalgaon Dam (Solapur, Maharashtra). It provides a **fully reproducible, cell-by-cell Jupyter notebook** that goes from raw hydraulic rasters and settlement shapefiles to a publish-ready Emergency Action Plan (EAP) with Annexure 3A/3B tables, hazard-zone maps, and evacuation priority rankings — in a single run.

> **For researchers:** The codebase is modular, scenario-agnostic, and directly portable to any dam in India or elsewhere. Every analytical decision is documented, every threshold is justified, and every output column is defined.

---

## Table of Contents

- [Study Area & Dam Profile](#study-area--dam-profile)
- [Analysis Overview](#analysis-overview)
- [Repository Structure](#repository-structure)
- [Methodology at a Glance](#methodology-at-a-glance)
- [Hazard Classification Schema](#hazard-classification-schema)
- [Key Outputs](#key-outputs)
- [Quick Start](#quick-start)
- [Input Data Requirements](#input-data-requirements)
- [Output Files Reference](#output-files-reference)
- [Assumptions & Limitations](#assumptions--limitations)
- [Reproducibility](#reproducibility)
- [Citation](#citation)
- [License](#license)

---

## Study Area & Dam Profile

| Parameter | Value |
|-----------|-------|
| **Dam Name** | Jawalgaon Dam |
| **District** | Solapur |
| **State** | Maharashtra, India |
| **River System** | *(Bhima basin tributary)* |
| **DEM Source** | Sentinel-30 m |
| **Hydraulic Model** | HEC-RAS / equivalent 2D raster output |
| **Scenarios Modelled** | Piping failure · Overtopping failure |
| **Overtopping Simulation Offset** | T₀ = 19.0 hours before breach |

The downstream floodplain encompasses agricultural land, villages, rural roads, and critical infrastructure (schools, hospitals, religious sites, water wells) that are systematically enumerated and risk-ranked by this tool.

---

## Analysis Overview

```
Raw Hydraulic Rasters (Depth, Velocity, WSE, Arrival Time)
           │
           ▼
  ┌─────────────────────────────────────────────────────┐
  │   22-Cell Sequential Jupyter Notebook Pipeline      │
  │                                                     │
  │  [1] Library install + [2] Config (single edit)     │
  │  [3] Input validation + [4] CRS auto-detection      │
  │  [5] Column inspection + [6] Polygon smoothing      │
  │  [7] Dam localisation + [8] Raster reprojection     │
  │  [9] Inundation polygons + [10] Arrival reclassify  │
  │  [11] H1–H6 hazard rasters + [12] OSM bulk query   │
  │  [13] Zonal statistics + [14] PAR calculation       │
  │  [15] Evacuation priority + [16] Shelter assignment │
  │  [17] Road/infrastructure analysis                  │
  │  [18] Shapefile export + [19] Annexure 3A/3B        │
  │  [20] Colour-coded maps + [21] Summary report       │
  │  [22] ZIP download                                  │
  └─────────────────────────────────────────────────────┘
           │
           ▼
   EAP-Standard Outputs (Shapefiles · CSVs · Maps · Report)
```

---

## Repository Structure

```
jawalgaon-dam-par-analysis/
│
├── 📓 notebooks/
│   └── Jawalgaon_PAR_Complete.ipynb   # Main analysis notebook (22 cells)
│
├── 📦 data/
│   ├── raw/
│   │   └── Jawalgaon_PAR_Complete.zip # All input shapefiles + raster outputs
│   └── outputs/                       # Generated by the notebook
│       ├── shapefiles/                # .shp layers: inundation, settlements, roads, POI
│       ├── csv/                       # PAR tables, Annexure 3A/3B, distance tables
│       └── rasters/                   # Hazard rasters (H1–H6), arrival-time rasters
│
├── 📄 docs/
│   ├── figures/                       # Maps, diagrams, flowcharts
│   ├── annexures/                     # EAP Annexure 3A & 3B (PDF/CSV)
│   ├── METHODOLOGY.md                 # Full analytical methodology
│   └── DATA_DICTIONARY.md             # All column/field definitions
│
├── 🧾 METHODOLOGY.md                  # (mirrored at root for discoverability)
├── 📖 DATA_DICTIONARY.md
├── 🤝 CONTRIBUTING.md
├── 📋 CHANGELOG.md
├── 📌 requirements.txt
├── 🚫 .gitignore
└── 📄 LICENSE
```

---

## Methodology at a Glance

> Full detail in [METHODOLOGY.md](METHODOLOGY.md)

### 1 · Coordinate Reference System
The notebook auto-detects the correct UTM zone from the settlement centroid and reprojects all inputs (rasters + vectors) to a consistent metric CRS before any area, distance, or overlay computation.

### 2 · Inundation Polygon Generation
Water-depth rasters (>0 m) are vectorised via `rasterio.features.shapes`, unioned, and optionally Gaussian-smoothed to produce per-scenario inundation extents.

### 3 · Arrival Time Reclassification
Raw simulation-clock arrival times are converted to **hours after breach (T=0)** by subtracting the user-specified offset (`OVT_OFFSET_HRS` for overtopping; auto-detected minimum for piping). This is critical for accurate evacuation window calculations.

### 4 · Hazard Classification (DV Product)
Each pixel is classified on the **Depth × Velocity** product (m²/s):

| Class | DV Range (m²/s) | Label |
|-------|-----------------|-------|
| H1 | 0.00 – 0.25 | Very Low |
| H2 | 0.25 – 0.50 | Low |
| H3 | 0.50 – 1.00 | Medium |
| H4 | 1.00 – 2.00 | High |
| H5 | 2.00 – 4.00 | Very High |
| H6 | > 4.00 | Extreme |

### 5 · PAR Calculation
Population at Risk (PAR) per settlement = Population × (Inundated Area / Total Area). Flood status labels (e.g., *VILLAGE SURROUNDED BY FLOOD*) are assigned based on inundation percentage and road-access analysis.

### 6 · Evacuation Priority Triage
Settlements are ranked P1–P5 based on arrival time (hours to flood wave):

| Code | Window | Action |
|------|--------|--------|
| P1 | 0–2 hours | IMMEDIATE |
| P2 | 2–4 hours | URGENT |
| P3 | 4–6 hours | HIGH PRIORITY |
| P4 | 6–8 hours | STANDARD |
| P5 | 8+ hours | PLANNED |

A composite **Vulnerability Index (0–1)** integrates hazard class, inundation percentage, PAR magnitude, and distance from dam.

### 7 · Infrastructure & Road Inundation
OpenStreetMap data is fetched in a single bulk Overpass API query (roads + all POI categories + bridges/canals/barrages). Each feature is tagged with flood arrival time, depth, and hazard class. Critical infrastructure (hospitals, schools, wells) is flagged for emergency prioritisation.

---

## Key Outputs

| Output File | Description |
|-------------|-------------|
| `Annexure_3A_Piping.csv` | EAP standard settlement table — piping scenario |
| `Annexure_3A_Overtopping.csv` | EAP standard settlement table — overtopping scenario |
| `Annexure_3B_Piping_PAR.csv` | PAR summary with evacuation priorities — piping |
| `Annexure_3B_Overtopping_PAR.csv` | PAR summary with evacuation priorities — overtopping |
| `Master_Distance_From_Dam.csv` | All features sorted by river chainage |
| `Critical_Infrastructure_At_Risk.csv` | Schools, hospitals, wells with hazard status |
| `Roads_Inundation.csv` | Road segments with flood arrival and depth |
| `Flood_Wave_Travel_Time.csv` | Wave front travel-time table |
| `Zonal_Statistics.csv` | Depth/velocity/WSE/arrival statistics per settlement |
| `inundation_piping.shp` | Flood extent polygon — piping |
| `inundation_overtopping.shp` | Flood extent polygon — overtopping |
| `settlements_piping.shp` | Settlements with full PAR attributes — piping |
| `settlements_overtopping.shp` | Settlements with full PAR attributes — overtopping |
| `hazard_piping.tif` | H1–H6 classified hazard raster — piping |
| `hazard_overtopping.tif` | H1–H6 classified hazard raster — overtopping |
| `arrival_piping_relative.tif` | Arrival time (hrs after breach) raster — piping |
| `arrival_overtopping_relative.tif` | Arrival time (hrs after breach) raster — overtopping |

---

## Quick Start

### Option A — Google Colab (Recommended, zero setup)
Click the badge at the top of this page, or:
```
https://colab.research.google.com/github/YOUR_USERNAME/jawalgaon-dam-par-analysis/blob/main/notebooks/Jawalgaon_PAR_Complete.ipynb
```
Upload your input rasters and shapefiles to `/content/`, then run all cells top-to-bottom.

### Option B — Local Installation

```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/jawalgaon-dam-par-analysis.git
cd jawalgaon-dam-par-analysis

# 2. Create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch Jupyter
jupyter notebook notebooks/Jawalgaon_PAR_Complete.ipynb
```

### ⚙️ Configuration (Cell 2 only)
Edit **only Cell 2** to point to your files:
```python
DAM_NAME        = "Jawalgaon Dam"
DAM_DISTRICT    = "Solapur"
DAM_STATE       = "Maharashtra"
OVT_OFFSET_HRS  = 19.0          # simulation clock hours at dam breach

PIPING = {
    "depth":        "/content/Depth (Max).Terrain.SENTINAL-30M-DEM.tif",
    "velocity":     "/content/Velocity (Max).Terrain.SENTINAL-30M-DEM.tif",
    "arrival_time": "/content/Arrival Time (Max).Terrain.SENTINAL-30M-DEM.tif",
    "wse":          "/content/WSE (Max).Terrain.SENTINAL-30M-DEM.tif",
}
# ... same pattern for OVERTOPPING
```

---

## Input Data Requirements

### Hydraulic Rasters (per scenario)
| Band | File suffix pattern | Unit |
|------|---------------------|------|
| Maximum water depth | `Depth (Max)…tif` | metres |
| Maximum flow velocity | `Velocity (Max)…tif` | m/s |
| Arrival time | `Arrival Time (Max)…tif` | hours (simulation clock) |
| Water surface elevation | `WSE (Max)…tif` | metres (MSL) |

### Shapefiles
| Layer | Required Columns | Purpose |
|-------|-----------------|---------|
| `village_polygon.shp` | `Name`, `Pop` | Settlement extents + population |
| `River.shp` | geometry | River centreline for chainage |
| `Submurgen.shp` | geometry | Reservoir submergence zone |
| `Watershed.shp` | geometry | Catchment boundary |
| `shelter locations_*.shp` | `Shelter` | Designated evacuation shelters |

> **Column names are configurable.** If your shapefile uses `village_name` instead of `Name`, update the `NAME_COLUMN` variable in Cell 2.

---

## Assumptions & Limitations

See [METHODOLOGY.md § Assumptions](METHODOLOGY.md#assumptions) for the complete list. Key points:

1. **Hydraulic inputs are taken as-is.** This notebook does not run a hydraulic model; it analyses outputs from HEC-RAS (or equivalent). Model accuracy governs result accuracy.
2. **Population data are census-period values.** Seasonal migration and day/night population variation are not modelled.
3. **OSM data completeness varies** by region. Sparse POI coverage in rural Maharashtra may undercount critical infrastructure.
4. **30 m DEM resolution** (Sentinel) limits accuracy for narrow valleys and small settlements; LiDAR would improve results.
5. **PAR = proportional area method.** It assumes uniform population distribution within each settlement polygon, which overestimates PAR in settlements with concentrated core areas near the flood fringe.

---

## Reproducibility

This notebook is designed for **full end-to-end reproducibility**:
- All random seeds are fixed where applicable.
- CRS detection is fully automated — no manual EPSG entry required.
- OSM queries are deterministic given the same bounding box and timestamp.
- All intermediate outputs are saved to disk so individual cells can be re-run independently.

To reproduce the exact Jawalgaon outputs, extract `data/raw/Jawalgaon_PAR_Complete.zip`, place the files in `/content/` (Colab) or update paths in Cell 2, and run all 22 cells.

---

## Citation

If you use this work in academic research, policy reports, or EAP documentation, please cite:

```bibtex
@misc{jawalgaon_par_2026,
  author       = {[Your Name]},
  title        = {Jawalgaon Dam Break — Population At Risk (PAR) Analysis},
  year         = {2026},
  publisher    = {GitHub},
  journal      = {GitHub Repository},
  howpublished = {\url{https://github.com/YOUR_USERNAME/jawalgaon-dam-par-analysis}},
  note         = {Dual-scenario GIS-based PAR assessment: Piping and Overtopping, Solapur, Maharashtra}
}
```

---

## Related Work & Standards

- **CWC (Central Water Commission) EAP Guidelines** — this tool produces Annexure 3A/3B compliant with the CWC format
- **NDMA Dam Safety Guidelines (2019)**
- **FEMA Inundation Mapping Guidelines** — H1–H6 hazard schema follows international practice
- **HEC-RAS 2D Hydraulic Modelling** — the upstream hydraulic model whose outputs this tool consumes

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

You are free to use, adapt, and redistribute this code for any dam safety or emergency planning application. Attribution is appreciated.

---

<div align="center">

*Built for dam safety practitioners, emergency managers, and geospatial researchers working on flood risk in South Asia and beyond.*

⭐ **Star this repository** if it helped your work · 🐛 [Report a Bug](.github/ISSUE_TEMPLATE/bug_report.md) · 💡 [Request a Feature](.github/ISSUE_TEMPLATE/feature_request.md)

</div>
