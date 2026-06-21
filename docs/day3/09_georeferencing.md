# Georeferencing in QGIS

Georeferencing is the process of assigning real-world coordinates to a raster dataset (such as a scanned paper map, a historical aerial photo, or an unreferenced topographic sheet). By defining geographic coordinates for specific pixels, GIS analysts can integrate historical data and paper maps directly into modern spatial analysis pipelines.

---

## 1. Core Georeferencing Concepts
To georeference a raster, you must align it with a known Coordinate Reference System (CRS) using reference control points.

* **Ground Control Points (GCPs):** Coordinates of identifiable features on both the unreferenced raster and a reference map (e.g., road intersections, river confluences, mountain peaks, or coordinate grid intersections).

* **Transformation Algorithms:** Mathematical models used to stretch, rotate, and scale the raster grid to match the target coordinates:
  * **Linear:** Performs simple scaling, rotation, and translation. Best for maps that are already orthorectified and only lack coordinate headers.
  * **Polynomial (1, 2, 3):** Applies complex polynomial equations to warp the image. Polynomial 1 (Affine) preserves parallel lines. Polynomial 2 and 3 stretch the image non-linearly to correct for paper folds, camera distortion, or terrain displacement.
  * **Thin Plate Spline (TPS):** Localized transformation that warps the image locally around GCPs. Highly effective for severely distorted historical maps.

* **Resampling Methods:** Algorithms used to calculate the new cell values of the output raster grid:
  * **Nearest Neighbor:** Assigns the value of the closest input pixel. Fastest method; preserves original discrete values (ideal for categorized maps).
  * **Bilinear Interpolation:** Computes a weighted average of the nearest 4 pixels. Smooths the output (ideal for continuous elevation datasets or satellite images).
  * **Cubic Convolution:** Computes a weighted average of the nearest 16 pixels. Sharpens edges but is computationally intensive.

---

## 2. Step-by-Step Georeferencing Workflow in QGIS
This workflow demonstrates how to georeference a scanned topographic basin map using a reference base map.

### Step 1: Open the Georeferencer Tool
* In QGIS Desktop, navigate to the main menu and select **Layer** > **Georeferencer**.
* In the Georeferencer window, click the **Open Raster** button (blue checkerboard icon) and select your scanned map file (e.g., `scanned_catchment_map.jpg`).

### Step 2: Set the Target CRS
* Click the **Transformation Settings** button (yellow gear icon).
* **Target CRS:** Select the projected coordinate system for your project area, such as **WGS 84 / UTM Zone 44N (EPSG:32644)** or **WGS 84 / UTM Zone 45N (EPSG:32645)**.

### Step 3: Add Ground Control Points (GCPs)
* Identify a clear, distinguishable landmark on the scanned map (e.g., a coordinate grid intersection or river junction).
* Select the **Add Point** tool (green dot with crosshair) and click precisely on that landmark.
* A dialog box will appear asking for coordinates. You can:
  1. Manually enter the **X (Easting)** and **Y (Northing)** coordinates printed on the scanned map's graticules grid.
  2. Click **From Map Canvas** and click on the same feature on your active QGIS base map layer to pull coordinates dynamically.
* Repeat this step to add **at least 4 to 6 GCPs** distributed evenly across the raster corners and center. Avoid clustering GCPs in a single region, as this causes distortion at the edges.

### Step 4: Configure Transformation Parameters
* Re-open **Transformation Settings** and configure:
  * **Transformation type:** Select **Polynomial 1** (or **Thin Plate Spline** if the paper map is folded).
  * **Resampling method:** Select **Bilinear**.
  * **Output raster:** Name your file `georeferenced_catchment_map.tif` (always save as GeoTIFF).
  * **Use 0 for transparency when needed:** Check this to hide black borders around the warped output raster.
  * **Load in QGIS when done:** Check this to view the output map directly on the canvas.
* Click **OK**.

### Step 5: Start the Transformation
* Click the **Start Georeferencing** button (green play icon) on the toolbar.
* The Georeferencer will warp the raster image and load `georeferenced_catchment_map.tif` onto your QGIS map canvas.

---

## 3. Residual Error Analysis
The Georeferencer table displays a **Residual** error (measured in pixels) for each GCP:

$$\text{Residual} = \sqrt{(X_{\text{actual}} - X_{\text{calculated}})^2 + (Y_{\text{actual}} - Y_{\text{calculated}})^2}$$

* **Mean Error (RMSE):** The average displacement error across all GCPs.
* **Interpretation:** A high residual error on a specific point indicates that the point was placed inaccurately or that the coordinate was mistyped. Use the **Move GCP Point** or **Delete Point** tool to adjust or recreate points with high residuals until the overall RMSE is minimized (ideally under $1.5$ to $2.0$ pixels).

---

## 4. Significance in Hydrology
Georeferencing plays an essential role in compiling historical and baseline data:
* **Historical Runoff Analysis:** Digitizing drainage networks and sub-basin boundaries from historical topographic maps to analyze decadal changes in stream alignment and channel shifting.
* **Siting Hydromet Stations:** Finding the exact coordinates of legacy gauge stations that were active before GPS coordinates were recorded, matching them with local coordinates written in historical commission reports.
* **Land-Use Trend Modeling:** Aligning old aerial photography layers to map forest cover dynamics and calculate runoff coefficient shifts over time.
