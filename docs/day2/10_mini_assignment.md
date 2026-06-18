# Mini-Assignment: Satellite Basin Data Acquisition and Metadata Registry

In this assignment, you will apply the search, filtering, and data discovery skills learned during Day 2 to download multispectral bands and digital elevation datasets for a target watershed in Nepal and construct a structured metadata catalog registry.

---

## 1. Objective

Download and compile a structured, analysis-ready remote sensing dataset for a selected river basin in Nepal (e.g., Kosi, Gandaki, Karnali, or local tributaries like Bagmati, West Rapti, Babai), and build a metadata spreadsheet registering the critical satellite characteristics of the acquired assets.

---

## 2. Selection of Target Basin

1. Select a river basin or sub-catchment in Nepal of interest to your work at WECS.

2. Identify its approximate geographic bounding box (coordinates in Decimal Degrees) or shape outline.

---

## 3. Step-by-Step Requirements

### Step 1: Sentinel-2 Multispectral Scene Search
Open Copernicus CDSE Browser or run a Python STAC API query:

1. Search for Sentinel-2 L2A imagery covering your target basin.

2. Set the search filter to a cloud cover threshold of $< 10\%$.

3. Find a dry-season scene (between November 2023 and April 2024) to ensure minimal snow cover and cloud interference.

4. Note down the unique scene **Product Identifier** (e.g., `S2A_MSIL2A_20231225...`).

### Step 2: Digital Elevation Model (DEM) Search
Open USGS EarthExplorer or CDSE Browser:

1. Search for digital elevation grids (SRTM 1 Arc-Second or Copernicus 30m DEM) covering your target basin coordinates.

2. Note down the individual tile names/identifiers covering the basin.

### Step 3: Downloading the Datasets
Download the following files to your structured local workspace:

1. For Sentinel-2, download the raw Green Band (Band 3) and NIR Band (Band 8) GeoTIFFs (either as individual bands via ObservEarth/STAC, or the full unzipped granule folder).

2. For the DEM, download the target tile GeoTIFFs.

3. Organize these files under:
   * `data/raw/sentinel2/`
   * `data/raw/dem/`

### Step 4: Compiling the Metadata Registry
Create a CSV or Excel spreadsheet named `basin_metadata_registry.csv` inside `data/metadata/`. The spreadsheet must contain a table with the following column headers and completed information:

| Dataset | Sensor/Platform | Date of Acquisition | Spatial Resolution (m) | Cloud Cover % | Coordinate Reference System (CRS) | File Size (MB) | Scene/Tile Identifier |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Sentinel-2 Green Band** | Sentinel-2A / MSI | `[YYYY-MM-DD]` | $10\text{ m}$ | `[Value]` | `[e.g., EPSG:32645]` | `[Value]` | `[Full Scene ID]` |
| **Sentinel-2 NIR Band** | Sentinel-2A / MSI | `[YYYY-MM-DD]` | $10\text{ m}$ | `[Value]` | `[e.g., EPSG:32645]` | `[Value]` | `[Full Scene ID]` |
| **Elevation Grid (DEM)** | Shuttle Radar / SRTM | `[February 2000]` | $30\text{ m}$ | $0\%$ | `[e.g., EPSG:4326]` | `[Value]` | `[Tile Name]` |

---

## 4. Submission Instructions

Compile your assignment outputs into a single ZIP archive named `Day2_Assignment_[YourName].zip` containing:

1. The compiled metadata registry spreadsheet (`data/metadata/basin_metadata_registry.csv` or `.xlsx`).

2. A screenshot of your structured local workspace folder tree showing the downloaded raw Sentinel-2 and DEM datasets in their respective directories.

3. A text file containing a brief paragraph explaining why the selected Sentinel-2 date and cloud cover thresholds are suitable for dry-season baseflow or water body mapping in your target basin.
