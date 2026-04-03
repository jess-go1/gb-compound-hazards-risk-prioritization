# GB Compound Hazards Risk Prioritization

> **Course:** Artificial Intelligence in Human Water Systems
> **Institution:** Technische Universität Berlin
> **Dataset:** ERA5 Reanalysis — Copernicus Climate Data Store (2009–2019)
> **Reference:** Tilloy et al. (2022), *Earth System Dynamics*, 13, 993–1020. https://doi.org/10.5194/esd-13-993-2022

---

## 📌 Research Question

> *For the regions in Great Britain that have experienced extreme compound-risk events, which categorizations could decision-makers define to prioritize the allocation of resources?*

---

## 🎯 Project Overview

This project detects, characterizes, and categorizes **compound hazard events** — the joint occurrence of extreme precipitation and extreme wind — across Great Britain using ERA5 climate reanalysis data for the period **2009–2019**.

Rather than analysing individual extreme events in isolation, we identify **spatiotemporal clusters** where both hazards co-occur. These compound events are then characterized using intensity, duration, spatial extent, and frequency metrics, which feed into a **risk prioritization table** designed to support resource allocation decisions by policymakers.

Regional analysis is performed at the **NUTS1 level**, covering the regions of Great Britain.

---

## 🔑 Key Results

| Metric | Value |
|---|---|
| Extreme precipitation observations | 36,925 |
| Extreme wind observations | 39,109 |
| Precipitation clusters (DBSCAN) | 161 |
| Wind clusters (DBSCAN) | 128 |
| Compound hazard events detected | 85 |
| Primary affected region | South West England |
| Season of events | SON (Sep–Nov 2019) |
| Average compound duration | 16.6 hours |
| Average peak wind gust | 24.13 m/s |
| Overall priority level | **HIGH** |

### Final Prioritization Table

| Region (NUTS1) | Total Events | High Risk | Avg Duration (h) | Avg Peak Wind (m/s) | Avg Peak Rain (mm) | Avg Extent (km²) | Priority |
|---|---|---|---|---|---|---|---|
| South West England | 73 | 25 | 16.6 | 24.13 | 4.67 | 35,043 | **HIGH** |

> South West England is consistent with Tilloy et al. (2022), who identified the southwestern coast as a primary hotspot for compound wind-precipitation clusters in Great Britain.

---

## 🔬 Methodology Pipeline

### Step 1 — Preprocessing: 99th Percentile Filtering
- Load ERA5 hourly precipitation and wind gust data (2009–2019)
- Compute **99th percentile threshold** per grid cell across all timesteps
- Retain only observations exceeding the threshold
- Output: 36,925 extreme precipitation and 39,109 extreme wind observations (~1% of total data)

> *The 99th percentile alone identifies extreme grid cells — DBSCAN identifies extreme events. A decision-maker needs event-level attributes (duration, extent, intensity), not grid cell statistics.*

### Step 2 — Clustering: DBSCAN
- Apply **DBSCAN** independently to extreme precipitation and wind observations
- Space-time cube representation: (normalized latitude x normalized longitude x hours)
- Parameters selected via k-NN distance plot (elbow method):
  - epsilon = 2.0 for both precipitation and wind
  - min_samples = 10
- Output: 161 precipitation clusters, 128 wind clusters

### Step 3 — Compound Hazard Detection
- Identify **spatiotemporal overlaps** between precipitation and wind clusters
- Compound event definition (following Tilloy et al., 2022):
  - **Spatial scale:** intersection of both cluster footprints (AND)
  - **Temporal scale:** union of both cluster durations (OR)
- Output: 85 compound hazard events

### Step 4 — Cluster Characterization
Each compound hazard cluster is described by:

| Attribute | Description |
|---|---|
| `compound_duration_h` | Total duration of the compound event (hours) |
| `compound_extent_km2` | Spatial footprint of the intersection (km²) |
| `peak_wind_ms` | Maximum wind gust within the cluster (m/s) |
| `peak_precipitation_mm` | Maximum precipitation accumulation (mm) |
| `nuts1_region` | NUTS1 region of Great Britain affected |
| `season` | Meteorological season (DJF/MAM/JJA/SON) |

### Step 5 — Risk Categorization and Prioritization Table
- Normalize intensities to empirical cumulative probability [0, 1]
- Classify events using a **3x3 wind x precipitation risk matrix** (High/Medium/Low)
- Aggregate by NUTS1 region
- Build final **decision-maker prioritization table**

---

## 📁 Project Structure

```
gb-compound-hazards-risk-prioritization/
├── data/
│   ├── raw/                          # ERA5 NetCDF files (not committed — too large)
│   │   ├── raindat_0919.nc           # Hourly precipitation 2009-2019
│   │   └── windat_0919.nc            # Hourly wind gust 2009-2019
│   ├── processed/                    # Intermediate outputs (not committed)
│   └── external/                     # Great Britain NUTS1 shapefiles
├── notebooks/
│   ├── 01_eda.ipynb                  # Exploratory data analysis
│   ├── 02_preprocessing.ipynb        # 99th percentile filtering + space-time cube
│   ├── 03_dbscan_clustering.ipynb    # DBSCAN on precipitation and wind
│   ├── 04_compound_detection.ipynb   # Spatiotemporal overlap detection
│   └── 05_risk_categorization.ipynb  # Risk table and prioritization
├── outputs/
│   ├── figures/                      # Generated maps and charts
│   └── reports/                      # Final prioritization table (CSV)
├── docs/
├── .gitignore
├── README.md
└── requirements.txt
```

---

## 🗂️ Data Source

- **Dataset:** ERA5 Hourly Reanalysis — ECMWF via Copernicus Climate Data Store
- **Access:** [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/)
- **Variables:**
  - `total_precipitation` — hourly accumulated precipitation (mm/h)
  - `instantaneous_10m_wind_gust` — hourly maximum wind gust at 10m (m/s)
- **Spatial resolution:** 0.25 x 0.25 degrees (53 x 41 grid cells)
- **Temporal coverage:** 2009–2019 (extreme event window: Sep–Nov 2019)
- **Study area:** Great Britain (NUTS1 regions)

> **Note on data structure:** The pre-processed ERA5 variable `p0005` contains full gridded values across all timesteps. Our own 99th percentile threshold was computed from this data to identify truly sparse extreme observations, consistent with the methodology of Tilloy et al. (2022).

---

## ⚙️ Setup

```bash
# 1. Clone the repository
git clone https://github.com/jess-go1/gb-compound-hazards-risk-prioritization.git
cd gb-compound-hazards-risk-prioritization

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add ERA5 data files to data/raw/
#    raindat_0919.nc and windat_0919.nc

# 4. Launch Jupyter and run notebooks in order (01 to 05)
jupyter notebook
```

---

## 🌿 Git Branching Strategy

```
main        <- stable, reviewed code only
develop     <- integration branch
feature/*   <- individual task branches
```

**Commit convention:**
```
feat:     new analysis or functionality
fix:      bug corrections
data:     data processing changes
viz:      figures and maps
docs:     README and documentation
refactor: code restructuring
```

---

## 📚 References

Tilloy, A., Malamud, B. D., and Joly-Laugel, A. (2022).
*A methodology for the spatiotemporal identification of compound hazards: wind and precipitation extremes in Great Britain (1979-2019).*
Earth System Dynamics, 13, 993-1020.
https://doi.org/10.5194/esd-13-993-2022

Zscheischler, J., Martius, O., Westra, S., et al. (2020).
*A typology of compound weather and climate events.*
Nature Reviews Earth and Environment, 1, 333-347.
https://doi.org/10.1038/s43017-020-0060-z

Hersbach, H., Bell, B., Berrisford, P., et al. (2020).
*The ERA5 global reanalysis.*
Quarterly Journal of the Royal Meteorological Society, 146(730), 1999-2049.
https://doi.org/10.1002/qj.3803

---

## 👤 Author

**Jessica Gonzalez Hurtado** (Matrikelnummer: 5406583)
*Artificial Intelligence in Human Water Systems — Technische Universitat Berlin, 2025/2026*
