# Notebooks

## `Jawalgaon_PAR_Complete.ipynb`

The single, complete analysis notebook. **Run cells top to bottom in sequence.**

### Cell Map

| Cell | Purpose | Key Output |
|------|---------|-----------|
| 1 | Install libraries | — |
| 2 | ✏️ **Configuration** ← *Edit this cell only* | Config variables |
| 3 | Validate all input files | Pass/fail checklist |
| 4 | Load shapefiles + auto-detect UTM CRS | GeoDataFrames in UTM |
| 5 | Inspect shapefile columns | Column name printout |
| 6 | Chaikin polygon smoothing | `settlements_smoothed.shp` |
| 7 | Dam location detection | `dam_location.shp` |
| 8 | Reproject rasters to UTM | `_reproj/*.tif` |
| 9 | Generate inundation polygons | `inundation_{sc}.shp` |
| 10 | Reclassify arrival time to T=0 | `arrival_{sc}_relative.tif` |
| 11 | H1–H6 hazard classification rasters | `hazard_{sc}.tif` |
| 12 | OSM bulk query (roads + POI) | In-memory GeoDataFrames |
| 13 | Zonal statistics per settlement | 16 raster statistics columns |
| 14 | PAR calculation + flood status | `{sc}_PAR`, `{sc}_flood_st` columns |
| 15 | Evacuation priority + vulnerability index | P1–P5 codes, VI scores |
| 16 | Nearest shelter assignment | `sh_name`, `sh_dist_km` columns |
| 17 | Road + infrastructure inundation | `Roads_Inundation.csv`, `Critical_Infrastructure_At_Risk.csv` |
| 18 | Save all shapefiles | Full shapefile suite |
| 19 | Build Annexure 3A & 3B | `Annexure_3A_*.csv`, `Annexure_3B_*.csv` |
| 20 | Colour-coded visualisations | Maps rendered inline |
| 21 | Executive summary report | Printed report |
| 22 | ZIP & download | `PAR_Outputs.zip` |

### Quick Config Reference (Cell 2)

```python
# ── Required edits ────────────────────────────────────────
DAM_NAME        = "Jawalgaon Dam"       # Your dam name
DAM_DISTRICT    = "Solapur"             # District
DAM_STATE       = "Maharashtra"         # State
OVT_OFFSET_HRS  = 19.0                  # Simulation hours at breach (overtopping)

# ── Raster file paths ─────────────────────────────────────
PIPING = {
    "depth":        "/content/Depth (Max).Terrain…tif",
    "velocity":     "/content/Velocity (Max).Terrain…tif",
    "arrival_time": "/content/Arrival Time (Max).Terrain…tif",
    "wse":          "/content/WSE (Max).Terrain…tif",
}
OVERTOPPING = { … }  # same keys, different file names

# ── Shapefile paths ───────────────────────────────────────
SHP_SETTLEMENTS = "/content/village_polygon.shp"
SHP_RIVER       = "/content/River.shp"
SHP_RESERVOIR   = "/content/Submurgen.shp"
SHP_WATERSHED   = "/content/Watershed.shp"
SHP_SHELTERS    = "/content/shelter locations_Jawalgaon dam failure.shp"

# ── Column names — edit to match your shapefile ───────────
NAME_COLUMN     = "Name"    # Village name column
POP_COLUMN      = "Pop"     # Population column
SHELTER_NAME_COL = "Shelter"
```

### Running in Google Colab

1. Open the notebook via the Colab badge in the README
2. Upload all raster files (`.tif`) and shapefiles (`.shp` + sidecar files) to `/content/`
3. Edit Cell 2 paths if needed
4. **Runtime → Run all**
5. Download the ZIP from Cell 22

### Runtime

Full pipeline on typical Colab GPU/CPU: approximately **8–15 minutes** depending on raster resolution, settlement count, and OSM query response time.
