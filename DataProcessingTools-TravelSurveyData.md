# Spatial Difference-in-Differences Study: Household Travel Survey (HTS) Data Processing Pipeline

This guide outlines the complete workflow and tools required to prepare and analyze the 2000, 2014, and 2019 Household Travel Surveys (HTS) for a Spatial Difference-in-Differences (DiD) evaluation of the Rea Vaya BRT system in Johannesburg.

---

## Overview

- **Objective**: Assess the causal impact of Rea Vaya BRT on socio-economic characteristics and travel patterns using a Spatial DiD approach.
- **Datasets**:
  - HTS 2000: Household, Person, Trip
  - HTS 2014: Household, Person, Trip, Attitude
  - HTS 2019: Household, Person (No trip data)
- **Treatment Definition**: Spatial access to BRT (e.g., 500m–2km buffer) → % population served or binary flag, attached via TAZ or household location.

---

## Tools by Stage

| Step | Task | Recommended Tools |
|------|------|-------------------|
| 0 | Setup, variable inventory | Excel / Google Sheets / Notion |
| 1 | File audit, variable inspection | Python (`pandas`) or R (`readxl`) |
| 2 | Cleaning each file per year | Python (`pandas`) or R (`dplyr`) |
| 3 | Merge Trip + Person + Household | Python (`merge`) or R (`left_join`) |
| 4 | Handle missing (e.g. 2019 trips) | Python / R (add synthetic trips if needed) |
| 5 | Spatial treatment mapping | ArcGIS / QGIS / GeoPandas |
| 6 | Stack into master panel | Python (`concat`) or R (`bind_rows`) |
| 7 | Aggregate to person / HH / TAZ | Python (`groupby`) or R (`group_by`) |
| 8 | Spatial joins (e.g. zone trends) | ArcGIS / QGIS / GeoPandas |
| 9 | DiD estimation (standard/staggered) | R (`fixest`, `did`) / Stata (`reghdfe`, `csdid`) / Python (`linearmodels`) |
| 10 | Pre-trend plots, maps | R (`ggplot2`) / Python (`seaborn`, `matplotlib`) |
| 11 | Export final data | `.parquet`, `.csv`, GitHub |
| 12 | Report writing | RMarkdown, LaTeX, Word, Jupyter |

---

## Workflow Summary

### 1. Clean & Standardize Each Year
- Standardize IDs: `HHID`, `PID`, `TRIPID`, `YEAR`
- Recode variables (e.g. gender, income)
- Add explicit weights: `HH_WEIGHT`, `PERSON_WEIGHT`, `TRIP_WEIGHT`
- Save cleaned files: `hts2000_person_clean.parquet`, etc.

---

### 2. Merge Within Each Year
- Trip + Person + Household → **Trip-Level Fact Table**
- Add attitude (2014) or create synthetic trips (2019)
- Save merged table: `hts2014_fact.parquet`

---

### 3. Attach Spatial Treatment
- Use ArcGIS/QGIS to overlay BRT buffers with TAZ/HH
- Export: `TAZ_ID`, `TREAT_FLAG`, `TREAT_INTENSITY`
- Join to trip-level fact tables

---

### 4. Stack Into Longitudinal Panel
```python
combined = pd.concat([hts2000, hts2014, hts2019])
combined.to_parquet("hts_trip_super.parquet")
```

---
### 5. Derive Analysis Units
Unit	Method	Weight
Trip	Use hts_trip_super.parquet as-is	TRIP_WEIGHT
Person	groupby(HHID, PID, YEAR) + summary	PERSON_WEIGHT
Household	groupby(HHID, YEAR) + summary	HH_WEIGHT
Zone	groupby(TAZ_ID, YEAR) + weighted means	Sum of HH/pop weights

---

### 6. Prepare for DiD Estimation
python
``
df["POST"] = (df["YEAR"] >= 2014).astype(int)
df["DID"] = df["TREAT_FLAG"] * df["POST"]
Cluster: TAZ_ID, HHID
``
Plot trends: mode share, travel time, car ownership

Optional: use staggered DiD (Sun & Abraham or Callaway & Sant’Anna)

---

### Output Checklist
| Output                             | Format                        |
| ---------------------------------- | ----------------------------- |
| Cleaned yearly files               | `.parquet` or `.csv`          |
| Merged trip-level superfile        | `hts_trip_super.parquet`      |
| Aggregated person/household panels | `.csv`                        |
| TAZ-level summaries                | `.csv`                        |
| DiD-ready panel                    | `.csv`, `.dta`, or `.parquet` |
| Plots, maps                        | `.png`, `.pdf`                |
| Scripts                            | `.py`, `.R`, `.ipynb`, `.do`  |
| Documentation                      | `README.md`, `.md`, `.pdf`    |

---

### Tips
- Prefer .parquet for large files.
- Use GitHub for code + documentation version control.
- Keep metadata for each variable in a .json or .md file.
- Use fixest::feols or did::att_gt for robust DiD estimation in R.

---

### References
- Callaway & Sant’Anna (2021) [Stata: csdid, R: did]
- Sun & Abraham (2021) [Staggered DiD]
- Scott Cunningham, Causal Inference: The Mixtape
- ArcGIS / QGIS Documentation


