# Contributing to Jawalgaon Dam Break PAR Analysis

Thank you for your interest in improving this tool. Contributions from dam safety engineers, GIS practitioners, hydrologists, and emergency management professionals are especially welcome.

---

## How to Contribute

### 🐛 Bug Reports
Use the [Bug Report template](.github/ISSUE_TEMPLATE/bug_report.md). Please include:
- The cell number that failed
- The error message (full traceback)
- Your Python and library versions (`pip show geopandas rasterio`)
- A description of your input data (CRS, file types, column names)

### 💡 Feature Requests
Use the [Feature Request template](.github/ISSUE_TEMPLATE/feature_request.md). Good candidates include:
- Support for additional hydraulic model output formats
- New hazard classification schemas
- Alternative PAR estimation methods (e.g., dasymetric mapping)
- Adapters for other national EAP formats (beyond CWC/India)

### 🔀 Pull Requests
1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature-name`
3. Make your changes
4. Update `CHANGELOG.md` with a summary
5. Open a pull request against `main`

All PRs should:
- Maintain backward compatibility with the existing Cell 2 configuration interface
- Include a comment block explaining any new analytical decision or threshold
- Not break the sequential cell execution order

---

## Porting to Another Dam

This notebook is designed to be portable. To adapt it for a different dam:

1. **Edit only Cell 2** — update file paths, column names, dam name/district, and `OVT_OFFSET_HRS`
2. **Run Cell 5 first** to confirm the column names in your shapefile
3. If your OSM region has sparse data, the `Critical_Infrastructure_At_Risk.csv` will be short — this is expected, not a bug

Please consider opening a PR with your adapted notebook (with data paths generalised) to build a library of dam-specific configurations.

---

## Code Style

- Follow PEP 8
- Use descriptive variable names (not `df2`, `gdf_x`)
- Add comments at every analytical decision point explaining *why*, not just *what*
- Keep each notebook cell focused on a single logical step

---

## Contact

For questions about dam safety methodology, CWC compliance, or EAP format requirements, open a Discussion rather than an Issue.
