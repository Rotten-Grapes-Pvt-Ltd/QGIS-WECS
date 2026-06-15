# Mini-Assignment: Generating a Catchment Water Mask

In this assignment, you will apply the skills learned during Day 2 to download multispectral bands, calculate water indexes, extract surface water features as vector shapes, and compile a map layout showing surface water distribution.

---

## 1. Objective
Extract surface water bodies within a target basin using Sentinel-2 bands, convert the output to vector shapes, and compile an $A4$ landscape map showing the water boundaries overlaying a hillshaded DEM.

---

## 2. Provided Datasets

* `/data/vector/catchment_boundary.gpkg` (Study basin boundary, EPSG:32645).

* `/data/raster/sentinel2/` (Bands 3 and 8, clipped to study area).

* `/data/raster/elevation_dem.tif` (Copernicus $30	ext{ m}$ DEM).

---

## 3. Step-by-Step Requirements

### Step 1: NDWI Calculation
Open QGIS and load the Sentinel-2 bands. Use the **Raster Calculator** to calculate the NDWI surface:
$$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$
Save the output raster as `basin_ndwi.tif`.

### Step 2: Extracting Water Pixels (Reclassification)
Reclassify `basin_ndwi.tif` into a binary raster where pixels $> 0.0$ are assigned a value of $1$ (Water) and pixels $\le 0.0$ are assigned a value of `NoData` (null). This filters out land pixels, leaving only the water bodies.

### Step 3: Vectorization (Raster to Vector Conversion)
Convert the reclassified binary water raster into vector polygons:

* Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)**.

* **Input Layer:** Reclassified binary raster.

* Save the output as a GeoPackage layer named `surface_water_polygons.gpkg`.

### Step 4: Map Layout and Composition

1. Open the **Print Layout** and set the page size to **A4 Landscape**.

2. Add the map canvas. In the Layers panel, stack your styled layers:

   * **Top Layer:** `surface_water_polygons.gpkg` styled with a solid blue fill.

   * **Middle Layer:** River vector lines.

   * **Bottom Layer:** Styled elevation DEM set to $50\%$ transparency overlaying a hillshade layer (to show 3D terrain relief).

3. Add the following map elements:

   * **Title Block:** "Surface Water Distribution Map - `[Basin Name]`".

   * **Legend:** Displaying "Extracted Water Bodies", "Rivers", and "Elevation (meters)".

   * **Scale Bar and North Arrow.**

   * **Coord Grid Overlay:** Coordinates displayed in UTM meters (Zone 45N).

---

## 4. Submission Instructions
Export your print layout as a PDF. Submit a ZIP archive containing:

1. Your exported map PDF (`Basin_Surface_Water_Map.pdf`).

2. The reprojected GeoPackage database file containing your vector water polygons (`surface_water_polygons.gpkg`).

3. Your QGIS project file (`Day2_Assignment.qgz`).
