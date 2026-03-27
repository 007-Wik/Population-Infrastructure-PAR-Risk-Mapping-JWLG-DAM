# Changelog

All notable changes to this project are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2026-03-22

### Added
- 22-cell sequential Jupyter notebook for complete PAR analysis
- Dual-scenario support: Piping and Overtopping failure modes
- Auto-UTM CRS detection from settlement centroid
- Chaikin corner-cutting polygon smoothing (Cell 6)
- H1–H6 hazard classification rasters using DV product (Cell 11)
- Single-request OSM bulk Overpass query for roads + POI (Cell 12)
- Zonal statistics per settlement for all 8 hydraulic bands × 2 scenarios (Cell 13)
- PAR calculation with flood status labels (Cell 14)
- Composite vulnerability index and P1–P5 evacuation priority (Cell 15)
- Nearest shelter assignment (Cell 16)
- Road and critical infrastructure inundation analysis (Cell 17)
- CWC EAP standard Annexure 3A and 3B tables (Cell 19)
- Colour-coded visualisations in Colab (Cell 20)
- Executive summary report (Cell 21)
- Alert zone classification (Zone I/II/III)
- Master distance-from-dam CSV sorted by river chainage
- Flood wave travel time table
- Critical infrastructure at-risk export

### Technical
- All outputs saved to `/content/PAR_Outputs/`
- Final ZIP archive created for one-click download
- Full reproducibility from raw rasters to EAP documents in a single notebook run
