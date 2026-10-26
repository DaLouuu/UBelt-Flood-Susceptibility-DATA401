# GIS-Based Flood Risk Assessment of Student Housing in Manila's University Belt
**DATA401 Final Map Project**  
*Authors:* Dana Louise Guillarte & Chrysille Grace So | *Date:* April 2026

---

## 📌 Project Overview
This project evaluates the flood susceptibility and road accessibility of student housing accommodations within Manila's **University Belt (U-Belt)** under a 100-year flood scenario. By synthesizing OpenStreetMap (OSM) spatial data with DOST-PAGASA / Project NOAH flood hazard models, the study identifies high-risk student residential clusters, quantifies physical vulnerability, and assesses whether severe flooding severs critical road connectivity between student accommodations and anchor universities.

---

## 🔬 Methodology & Workflow

The pipeline is organized into three sequential Jupyter Notebooks:

### 1. `01.5_bounding_box.ipynb` (Study Area & Extent Justification)
- Queries administrative district boundaries (Sampaloc, Quiapo, San Miguel) via OSMnx.
- Establishes the study bounding box: `[14.5750, 14.6100]` N, `[120.9850, 121.0150]` E (~3.9 km × 3.2 km).
- Validates coverage across major anchor universities (UST, PUP, PLM, DLSU, Mapúa, FEU, CEU, San Sebastian) while excluding peripheral districts like Makati.
- Outputs `outputs/bbox_justification_map.html`.

### 2. `01_osm_data_pull.ipynb` (OSM Data Collection & Preprocessing)
- **Anchor Extraction:** Extracts university and college locations (`amenity=university|college`).
- **Student Housing Filter:** Queries dormitories, student accommodations, apartments, and residential structures within an **800m pedestrian buffer** of anchor universities.
- **Vulnerability Indicators:**
  - `building:levels`: Floor count / structural height (missing values imputed using dataset median).
  - `dist_waterway_m`: Proximity to natural waterways, canals, and esteros.
  - `vuln_type`: Vulnerability scoring by building type (dormitories prioritized).
  - `footprint_area_m2`: Footprint area derived from polygon geometry.
- **Exposure Indicator:** Generates a 200m × 200m spatial grid to compute localized building density.
- **Hazard Overlay:** Clips the Project NOAH 100-year return period flood hazard shapefile (`MetroManila_Flood_100year.shp`) and overlays it onto the housing dataset.
- Saves preprocessed datasets to `Data/processed/student_housing_final.geojson`.

### 3. `02_composite_risk_index.ipynb` (Risk Index, Hotspot & Accessibility Modeling)
- **Composite Flood Risk Index:** Normalizes indicators and executes a multi-criteria weighted overlay:
  - Flood Hazard Level: **40%**
  - Building in Flood Zone (Binary): **15%**
  - Building Density: **15%**
  - Floor Count / Levels: **15%**
  - Building Type: **10%**
  - Waterway Proximity: **5%**
  - Classifies structures into *Low Risk*, *Medium Risk*, and *High Risk*.
- **Spatial Hotspot Analysis:**
  - Kernel Density Estimation (KDE) clustering of high-risk buildings.
  - Local Getis-Ord $G_i^*$ spatial statistics across 200m grid cells (identifying statistically significant hot spots at 95% confidence, $z > 1.96$).
- **Network Accessibility Under Flood Simulation:**
  - Constructs a drivable road network graph using OSMnx/NetworkX (3,599 road segments).
  - Simulates severe flooding by removing road segments intersecting medium and high flood zones (2,946 segments impacted).
  - Computes shortest-path reachability from high-risk student accommodations to university anchor nodes to determine severed/cut-off housing.
- Generates static cartographic figures and the interactive web map.

---

## 🌐 Frontend & Visualization

### Does this repository have a frontend?
- **Interactive Web Frontend Deliverables:**
  - **`outputs/interactive_map.html`** (8.78 MB): A fully interactive, client-side Leaflet.js / Folium web map featuring:
    - Layer toggles (Flood Hazard Zones, Student Housing Risk Classes, Flooded Roads, Universities, Cut-off Housing).
    - Dynamic tooltips displaying composite risk score, floor count, and hazard classification for each building.
  - **`outputs/bbox_justification_map.html`** (19.7 KB): Interactive map demonstrating bounding box coverage and target institutions.
  - *How to view:* Double-click either `.html` file to open directly in any web browser, or serve locally via:
    ```bash
    python -m http.server 8000
    ```
    Then navigate to `http://localhost:8000/outputs/interactive_map.html`.
- **Static Map Deliverables (`outputs/`):**
  - `map_composite_risk.png`: Composite risk classification per building.
  - `map_hazard_exposure.png`: Flood hazard zone intersection.
  - `map_road_accessibility.png`: Normal vs. flooded road network accessibility.
  - `hotspot_kde.png`: KDE heat density surface.
  - `hotspot_gistar.png`: Getis-Ord $G_i^*$ hotspot clusters.
- **QGIS Cartography (`QGIS/`):**
  - Complete project file `QGIS_mapping.qgz` with print layouts and published vector maps in PDF format.

---

## 📂 Project Directory Structure

```text
UBelt-Flood-Susceptibility-DATA401/
│
├── notebooks/                      # Sequential analytical notebooks
│   ├── 01.5_bounding_box.ipynb     # Bounding box determination and sanity check
│   ├── 01_osm_data_pull.ipynb      # OSM data ingestion, cleaning, and hazard intersection
│   └── 02_composite_risk_index.ipynb # Risk index calculation, hotspot analysis, network routing
│
├── requirements.txt                # Environment dependencies
├── README.md                       # Project documentation
│
├── Data/
│   ├── Metro Manila/               # Project NOAH 100-year flood hazard shapefiles (.shp, .dbf, .prj)
│   ├── raw/                        # Raw OSM extracts (roads, universities, housing, waterways)
│   └── processed/                  # Cleaned GeoJSON layers (risk scores, flooded roads, grid)
│
├── outputs/
│   ├── interactive_map.html        # Interactive Folium web map
│   ├── bbox_justification_map.html # Bounding box interactive map
│   └── *.png                       # High-resolution output maps (KDE, Gi*, accessibility, risk)
│
└── QGIS/
    ├── QGIS_mapping.qgz            # QGIS mapping project
    ├── map_layout_template.qpt     # Print layout template
    └── *.pdf                       # High-resolution PDF exports
```

---

## 🚀 Setup & Execution

### Prerequisites
- Python 3.10+ (tested with Python 3.14)

### Installation
```bash
pip install -r requirements.txt
```

### Running the Notebooks
Execute the notebooks in sequence from either the root folder or within `notebooks/`:
1. `notebooks/01.5_bounding_box.ipynb`
2. `notebooks/01_osm_data_pull.ipynb`
3. `notebooks/02_composite_risk_index.ipynb`