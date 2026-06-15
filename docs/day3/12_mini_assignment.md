# Mini-Assignment: Building a Hydrological GIS Database

In this assignment, you will apply the skills learned during Day 3 to clean vector datasets, execute spatial joins, calculate catchment statistics, and compile a formatted layout map.

---

## 1. Objective
Build a structured hydrological database inside a single GeoPackage, calculate sub-basin average elevations and river lengths, and export a layout map illustrating river buffers and district statistics.

---

## 2. Datasets Provided

* `/data/vector/nepal_districts.gpkg` (District polygons).

* `/data/vector/river_network.gpkg` (Stream centerlines).

* `/data/raster/terrain_dem.tif` (Digital elevation model).

---

## 3. Step-by-Step Requirements

### Step 1: Database Creation
Create a new GeoPackage database file named `Catchment_Database.gpkg` in your vector data directory. Import all raw vector layers into this database.

### Step 2: Stream Buffer Analysis
Generate a $250	ext{ m}$ buffer around all stream networks. Dissolve the overlapping buffers to create a continuous riparian corridor layer.

### Step 3: Zonal Statistics
Run the **Zonal Statistics** tool using your elevation DEM to calculate the average terrain height for each district boundary polygon. Store the output columns as new fields inside `Catchment_Database.gpkg`.

### Step 4: Map Layout Composition

1. Set page layout to **A4 Landscape**.

2. Add map canvas containing:

   * **Background:** Styled DEM (Pseudocolor) overlaying a hillshade layer.

   * **Middle Layer:** Riparian buffer corridors (styled in semi-transparent green).

   * **Top Layer:** District boundaries styled graduated by their calculated average elevation values.

3. Add title blocks, legends, dynamic scale bars, north arrows, and coordinate graticules.

---

## 4. Submission Instructions
Export layout as PDF. Submit a ZIP containing:

1. Exported PDF map (`Catchment_GIS_Database_Map.pdf`).

2. Your GeoPackage database file (`Catchment_Database.gpkg`).

3. Your QGIS project file (`Day3_Assignment.qgz`).
