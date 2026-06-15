# Mini-Assignment: Watershed Terrain Characterization

In this assignment, you will apply the skills learned during Day 4 to preprocess a DEM, delineate a watershed, extract its stream network, and calculate basin properties.

---

## 1. Objective
Delineate a watershed from a pour point, calculate catchment characteristics (slope, aspect, stream order), and compile an $A4$ landscape layout map showing the outputs.

---

## 2. Datasets Provided

* `/data/raster/raw_dem.tif` (Raw elevation model).

* `/data/vector/outlet_point.gpkg` (Pour point vector layer).

---

## 3. Step-by-Step Requirements

### Step 1: DEM Conditioning
Open QGIS, import `raw_dem.tif`, and run the SAGA **Fill Sinks** tool. Save the output as `conditioned_dem.tif`.

### Step 2: Flow Routing
Run SAGA **Catchment Area** on `conditioned_dem.tif` to generate flow direction (`flow_direction.tif`) and flow accumulation (`flow_accumulation.tif`) layers.

### Step 3: Delineate Watershed
Run the SAGA **Upslope Area** tool using `flow_direction.tif` and the coordinate coordinates of your pour point inside `outlet_point.gpkg`. Convert the resulting upslope raster to a vector polygon and save it as `watershed_polygon.gpkg`.

### Step 4: Stream Ordering
Run SAGA **Stream Order** on your stream network, using the Strahler stream ordering system. Style the output vector stream segments by order (thicker lines for higher orders).

### Step 5: Map Layout Composition

1. Open the **Print Layout** and set the page size to **A4 Landscape**.

2. Add the map canvas containing:

   * **Top Layer:** Vector streams styled by Strahler Stream Order.

   * **Middle Layer:** Watershed boundary polygon styled with a red border.

   * **Bottom Layer:** Slope map (pseudocolor) overlaying a hillshade layer.

3. Add legends, titles, scale bars, north arrows, coordinate grid lines, and a text block detailing catchment statistics (area, average slope, maximum elevation).

---

## 4. Submission Instructions
Export layout as PDF. Submit a ZIP containing:

1. Exported PDF map (`Basin_Terrain_Characterization.pdf`).

2. Your QGIS project file (`Day4_Assignment.qgz`).

3. Your GeoPackage database file containing the watershed boundary polygon and styled stream networks (`watershed_polygon.gpkg`).
