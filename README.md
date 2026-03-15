# GB Compound Hazards Risk Prioritization

> **Course:** AI for Human Water Systems  
> **Dataset:** ERA5 Reanalysis — Copernicus Climate Data Store (2009–2019)  
> **Reference:** Tilloy et al. (2022), *Earth System Dynamics*, 13, 993–1020. https://doi.org/10.5194/esd-13-993-2022

---

## 📌 Research Question

> *For the regions in Great Britain that have experienced extreme compound-risk events, which categorizations could decision-makers define to prioritize the allocation of resources?*

---

## 🎯 Project Overview

This project detects, characterizes, and categorizes **compound hazard events** — the joint occurrence of extreme precipitation and extreme wind — across Great Britain using ERA5 climate reanalysis data for the period **2009–2019**.

Rather than analysing individual extreme events in isolation, we identify **spatiotemporal clusters** where both hazards co-occur. These compound events are then characterized using intensity, duration, spatial extent, and frequency metrics, which feed into a **risk prioritization table** designed to support resource allocation decisions by policymakers.

Regional analysis is performed at the **NUTS1 level**, covering the 9 regions of Great Britain:

| NUTS1 Code | Region |
|---|---|
| UKC | North East England |
| UKD | North West England |
| UKE | Yorkshire and The Humber |
| UKF | East Midlands |
| UKG | West Midlands |
| UKH | East of England |
| UKI | London |
| UKJ | South East England |
| UKK | South West England |
| UKL | Wales |
| UKM | Scotland |

---

## 🔬 Methodology Pipeline

### Step 1 — Preprocessing: Extreme Event Filtering
- Load ERA5 hourly precipitation and wind gust data for Great Britain (2009–2019)
- Compute **99th percentile thresholds** per grid cell for both variables
- Filter dataset to retain only observations exceeding thresholds
- Output: top 1% extreme events for precipitation and wind separately

> *Purpose: reduce the dataset from millions of hourly grid values to only high-impact climate observations, making DBSCAN computationally tractable.*

### Step 2 — Clustering: DBSCAN
- Apply **DBSCAN** (Density-Based Spatial Clustering of Applications with Noise) independently to:
  - Extreme precipitation events
  - Extreme wind gust events
- Clustering operates in a **space-time cube** (latitude × longitude × time)
- Key DBSCAN parameters:
  - `ε` (neighbourhood radius): determined via k-NN distance plot
  - `min_samples` (μ): minimum points to form a cluster

> *Purpose: group dispersed extreme-value grid points into coherent spatiotemporal events with measurable properties. The 99th percentile alone identifies extreme grid cells — DBSCAN identifies extreme events, enabling the calculation of duration, spatial extent, and density per event.*

### Step 3 — Compound Hazard Detection
- Identify **spatiotemporal overlaps** between precipitation clusters and wind clusters
- A compound hazard event is defined as:
  - **Spatial scale:** intersection of both clusters' footprints (AND)
  - **Temporal scale:** union of both clusters' durations (OR)
- Output: compound hazard cluster database for Great Britain (2009–2019)

### Step 4 — Cluster Characterization
Each compound hazard cluster is described by:

| Attribute | Description |
|---|---|
| `duration_h` | Total duration of the compound event (hours) |
| `spatial_extent_km2` | Spatial footprint of the compound cluster (km²) |
| `peak_wind_gust` | Maximum wind gust within the cluster (m/s) |
| `max_precipitation` | Maximum precipitation accumulation within the cluster (mm) |
| `density` | Number of extreme grid points per cluster |
| `start_date` | Start datetime of the event |
| `nuts1_region` | NUTS1 region(s) of Great Britain affected |

### Step 5 — Risk Categorization & Prioritization Table
- Classify each compound cluster into **risk levels** (High / Medium / Low) using a wind intensity × precipitation intensity risk matrix
- Aggregate results by NUTS1 region across Great Britain
- Build final **decision-maker prioritization table** per region including:
  - Event frequency
  - Average duration
  - Average intensity (wind + precipitation)
  - Average spatial extent
  - Overall risk category

---

## 📁 Project Structure

```
gb-compound-hazards-risk-prioritization/
├── data/
│   ├── raw/                          # ERA5 original NetCDF files (not committed)
│   │   ├── era5_precipitation/       # Hourly precipitation 2009-2019
│   │   └── era5_wind/                # Hourly wind gust 2009-2019
│   ├── processed/                    # Cleaned and filtered data (not committed)
│   └── external/                     # Great Britain NUTS1 shapefiles
├── notebooks/
│   ├── 01_eda.ipynb                  # Exploratory data analysis
│   ├── 02_preprocessing.ipynb        # 99th percentile filtering
│   ├── 03_dbscan_clustering.ipynb    # DBSCAN on precipitation and wind
│   ├── 04_compound_detection.ipynb   # Spatiotemporal overlap detection
│   └── 05_risk_categorization.ipynb  # Risk table and prioritization
├── src/
│   ├── __init__.py
│   ├── preprocessing.py              # Percentile filtering functions
│   ├── clustering.py                 # DBSCAN implementation and helpers
│   ├── compound.py                   # Compound hazard overlap logic
│   └── visualization.py             # Maps and plots
├── outputs/
│   ├── figures/                      # Generated maps and charts
│   └── reports/                      # Summary tables and reports
├── docs/                             # Additional documentation
├── .gitignore
├── README.md
└── requirements.txt
```

---

## 🗂️ Data Source

- **Dataset:** ERA5 Hourly Reanalysis — European Centre for Medium-Range Weather Forecasts (ECMWF)
- **Access:** [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/)
- **Variables:**
  - `total_precipitation` — hourly accumulated precipitation (mm/h)
  - `instantaneous_10m_wind_gust` — hourly maximum wind gust at 10m (m/s)
- **Spatial resolution:** 0.25° × 0.25°
- **Temporal coverage used:** 2009–2019
- **Study area:** Great Britain (NUTS1 regions)

---

## ⚙️ Setup

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/gb-compound-hazards-risk-prioritization.git
cd gb-compound-hazards-risk-prioritization

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch Jupyter
jupyter notebook
```

---

## 🌿 Git Branching Strategy

```
main        ← stable, reviewed code only
develop     ← integration branch for both contributors
feature/*   ← individual task branches (e.g. feature/dbscan-clustering)
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

## 📚 Reference

Tilloy, A., Malamud, B. D., and Joly-Laugel, A. (2022).
*A methodology for the spatiotemporal identification of compound hazards: wind and precipitation extremes in Great Britain (1979–2019).*
Earth System Dynamics, 13, 993–1020.
https://doi.org/10.5194/esd-13-993-2022

---

## 👥 Authors

- [Your Name]
- [Partner's Name]  
*AI for Human Water Systems — 2024/2025*
