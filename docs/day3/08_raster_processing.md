# Raster Processing and Map Algebra

Rasters represent continuous geographic surfaces (such as elevation, temperature, or spectral indices) using a grid of cells (pixels). Unlike vector shapes, raster processing is performed on a cell-by-cell basis across grid rows and columns. This section covers map algebra, raster warping/resampling, reclassification, and raster-vector conversions.

We will use the localized Digital Elevation Model (DEM) dataset:

*   **DEM Dataset:** `output_hh.tif` (elevation values ranging from $600$ to $7000$ meters)
    *   **Local Project File:** [docs/data/Natural_Earth_quick_start/DEM/output_hh.tif](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/output_hh.tif)

---

## 1. Raster Structure and Cell Concepts

A raster grid consists of columns and rows defined by spatial coordinate parameters:

```text
    RASTER CELL LAYOUT (DEM EXAMPLE)
    +-------+-------+-------+
    | 650.5 | 682.2 | 710.8 |  <-- Cells containing elevation values (meters)
    +-------+-------+-------+
    | 640.1 | 674.9 | 705.2 |  
    +-------+-------+-------+
    | 632.0 | -9999 | 698.8 |  <-- -9999 represents NoData (null) cells
    +-------+-------+-------+
    [Cell Size: 30m x 30m]
```

*   **Continuous vs. Categorical Rasters:**
    
    *   *Continuous Rasters* represent surfaces where values change gradually over space without abrupt breaks (e.g., elevation, temperature, slope, or spectral indices like NDVI).
    
    *   *Categorical Rasters* represent discrete thematic classes or groups (e.g., land cover classes, soil types, or administrative zones where cell values are integers mapping to a lookup table).

*   **Cell Size (Resolution):** The geographic width and height of a single pixel (e.g., $30\text{ m}$ for the ALOS PALSAR / SRTM grids). All calculations inside a cell are averaged across this area.

*   **NoData Value:** A designated placeholder number (e.g., `-9999` or `-3.402823e+38`) indicating the absence of data. Operations on a NoData cell always output NoData.

*   **Bands:** A single raster file can contain multiple stacked layers (bands). Satellite scenes contain separate spectral bands (Red, Green, Blue, Near-Infrared), whereas elevation DEMs are singleband rasters.

---

## 2. Raster Mosaicking, Clipping, and Virtual Rasters

Before performing spatial calculations, raw raster data tiles must be compiled, merged, and clipped to the exact project boundary.

### Exercise 1: Merge Adjacent DEM Tiles (Mosaicking)

When a study catchment spans across multiple tile boundaries, tiles must be merged into a single continuous raster grid.

1. Locate adjacent DEM tiles (e.g., `tile_east.tif` and `tile_west.tif` inside your workspace directory).

2. Go to **Raster** > **Miscellaneous** > **Merge**.

3. Set the parameters:

   * **Input layers:** Click `...` and check the box for both DEM tiles.

   * **Output data type:** Select **Float32** (preserves decimal elevation values).

   * **Merged:** Save the output grid as `mosaicked_dem.tif`.

4. Click **Run**. The output is a single seamless elevation layer.

### Exercise 2: Clip DEM to River Basin (Clip by Mask Layer)

Trim a large raster grid down to the exact boundary of a vector polygon to minimize processing times.

1. Load `output_hh.tif` (continuous DEM) and `ne_10m_admin_0_countries` (polygon layer).

2. Filter the country layer for Nepal: Open **Select by Expression** (`Ctrl+F3`) on the country layer and select using: `"NAME" = 'Nepal'`.

3. Go to **Raster** > **Extraction** > **Clip Raster by Mask Layer**.

4. Set the parameters:

   * **Input Layer:** `output_hh.tif` (the grid to cut).

   * **Mask Layer:** `ne_10m_admin_0_countries` (check **Selected features only**).

   * **Source CRS / Target CRS:** Match both to the input CRS (EPSG:4326).

   * Keep the box **Match the resolution of the input raster** checked.

   * **Clipped (mask):** Save the output as `nepal_dem_clipped.tif`.

5. Click **Run**. The resulting raster will follow Nepal's exact national border, with areas outside the boundary set to NoData.

---

## 3. Map Algebra and the Raster Calculator

The **Raster Calculator** (**Raster** > **Raster Calculator**) evaluates mathematical or logical expressions on one or more raster grids on a cell-by-cell basis:

$$\text{Output Cell} = f(\text{Input Cell}_A, \text{Input Cell}_B)$$

### Exercise 3: Extract High-Altitude Zones (Binary Masking)

Isolate all mountainous terrain higher than $4000\text{ meters}$ in `output_hh.tif`.

1. Open **Raster** > **Raster Calculator**.

2. Under **Raster Bands**, double-click `"output_hh@1"` to add it to the expression box.

3. Complete the expression to check for values greater than 4000:

   ```sql
   "output_hh@1" > 4000
   ```

4. Set the **Output layer** to save inside your project output folder as `high_altitude_mask.tif`.

5. Ensure the Spatial Reference System matches `output_hh.tif`. Click **OK**.

6. The output is a binary grid where pixels are assigned $1$ (elevations $> 4000\text{ m}$) or $0$ (elevations $\le 4000\text{ m}$).

### Exercise 4: Extract Altitudinal Valleys (Multi-Criteria Masking)

Extract middle-altitude valleys and hills ranging strictly between $1500$ and $3500\text{ meters}$.

1. Open the **Raster Calculator**.

2. Enter the following logical combination expression:

   ```sql
   ("output_hh@1" >= 1500) * ("output_hh@1" <= 3500)
   ```

   *(Multiplying these boolean statements acts as a logical AND operation—only cells where both conditions are true ($1 \times 1$) will output $1$.)*

3. Save the output as `valley_hills_mask.tif` and click **OK**.

---

## 4. Raster Warping and Resampling Algorithms

When running analysis combining multiple rasters, they must share the exact same cell sizes, extents, and spatial alignments.

*   **Raster Warping (Reprojection):** Changes the Coordinate Reference System (CRS) of a raster grid. Access via **Raster** > **Projections** > **Warp (Reproject)**.

*   **Resampling:** Injected during reprojection or cell resizing to calculate new pixel values from the old grid layout. Selecting the wrong resampling algorithm will corrupt your data:

| Resampling Method | Ideal Data Type | Calculation Logic | Hydrological Impact |
| :--- | :--- | :--- | :--- |
| **Nearest Neighbor** | Categorical / Discrete | Copies the value of the nearest cell. | Preserves class values (e.g., landuse codes $1, 2, 3$). Using it on continuous elevation data creates blocky steps. |
| **Bilinear** | Continuous | Calculates a linear average of the 4 nearest pixels. | Smooths values. Ideal for continuous rasters like elevation grids (DEMs) or temperature surfaces. |
| **Cubic Convolution** | Continuous | Fits a smooth curve across the 16 nearest pixels. | High precision, but computationally heavy. Best for high-resolution imagery and ortho-rectification. |
| **Lanczos** | Continuous | Applies a sinc windowed filter across 36 neighboring pixels. | High detail retention, used for high-fidelity resampling of multi-spectral satellite imagery. |

### Exercise 5: Reproject and Resample the DEM

Reproject the elevation grid from a geographic coordinate system to a metric coordinate system for accurate slope calculations.

1. Go to **Raster** > **Projections** > **Warp (Reproject)**.

2. Set the parameters:

   * **Input Layer:** `output_hh.tif` (EPSG:4326).

   * **Target CRS:** Select **WGS 84 / UTM Zone 45N (EPSG:32645)** (metric UTM projection).

   * **Resampling method to use:** Select **Bilinear** (vital for continuous elevation data to prevent stepping artifacts).

   * **Output file resolution:** Enter `30` (forces the output pixels to be exactly $30\text{ m} \times 30\text{ m}$).

   * **Reprojected:** Save the file as `output_hh_utm_30m.tif`.

3. Click **Run**. The output raster is now projected in meters, ready for catchment slope derivations.

---

## 5. Raster Analysis: Extracting Terrain Derivatives

QGIS uses GDAL algorithms to calculate topographic variables from elevation grids. Slope and aspect calculations are geometry-dependent and must be run on projected DEMs (in meters) to avoid unit mismatch errors.

### Exercise 6: Calculate Terrain Slope and Aspect

Extract slope angles to locate high-velocity runoff zones.

1. Go to **Raster** > **Analysis** > **Slope**.

2. Set the parameters:

   * **Input Layer:** `output_hh_utm_30m.tif` (our projected DEM).

   * Keep the box **Slope expressed as percent instead of degrees** unchecked (we want degrees).

   * **Slope:** Save as `dem_slope_degrees.tif`.

3. Click **Run**. Output values range from $0^\circ$ (flat plains) to $>45^\circ$ (steep mountain walls).

4. Repeat the process for **Raster** > **Analysis** > **Aspect**:

   * **Aspect:** Save output as `dem_aspect.tif`.

   * Aspect values indicate orientation in degrees ($0^\circ$ = North, $90^\circ$ = East, $180^\circ$ = South, $270^\circ$ = West).

### Exercise 7: Generate Hillshade for Visualization

Create a simulated 3D shaded relief layer to assist in visual catchment mapping.

1. Go to **Raster** > **Analysis** > **Hillshade**.

2. Set the parameters:

   * **Input Layer:** `output_hh_utm_30m.tif`.

   * **Azimuth (light source angle):** `315` (North-West light is the cartographic standard).

   * **Altitude (elevation angle of light):** `45`.

   * **Hillshade:** Save output as `dem_hillshade.tif`.

3. Click **Run**. Place this layer under your elevation layer in QGIS, set the elevation layer's transparency to 40%, and select a colorful pseudocolor ramp to create a beautiful 3D terrain effect.

---

## 6. Raster Reclassification

Reclassification groups continuous cell values into simplified, discrete categories (converting continuous rasters to categorical rasters).

*   **Accessing:** Open the Processing Toolbox and select **Raster Analysis** > **Reclassify by Table**.

### Exercise 8: Reclassify DEM into Altitudinal Zones

Classify the continuous elevation raster `output_hh.tif` into three simple ecological zones: Lowland, Mid-hills, and Highland.

1. Search for **Reclassify by Table** in the Processing Toolbox.

2. Set the **Input raster** to `output_hh.tif`.

3. Under **Reclassification table**, click `...` to open the table editor. Add three rows:

   | Minimum | Maximum | Value |
   | :--- | :--- | :--- |
   | 600 | 2000 | 1 |
   | 2000 | 4500 | 2 |
   | 4500 | 7000 | 3 |

4. Check **Use no data when no range matches**.

5. Save the output raster as `elevation_ecological_zones.tif` and click **Run**.

6. The output will contain only three pixel values ($1$, $2$, and $3$), representing the three zones.

---

## 7. Raster-Vector Conversions

In spatial analysis workflows, you often need to transfer data between vector formats and raster structures:

### Raster to Vector (Polygonize)

Converts a raster grid into vector polygons. Adjacent pixels with identical integer values are merged into single polygon boundaries.

*   **Accessing:** Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)**.

*   **Exercise 9:** Convert the binary high-altitude raster mask created in Section 3 into vector polygons.
    
    1. Open **Raster** > **Conversion** > **Polygonize**.
    
    2. **Input Layer:** Select `high_altitude_mask.tif` (the binary mask).
    
    3. **Name of the field to create:** Enter `altitude_class` (stores values $1$ or $0$).
    
    4. **Vectorized:** Save as a GeoPackage table named `high_altitude_peaks`.
    
    5. Click **Run**. Select the polygons where `altitude_class = 1` to isolate the peaks.

### Vector to Raster (Rasterize)

Converts vector geometries (points, lines, or polygons) into a grid of raster cells.

*   **Accessing:** Go to **Raster** > **Conversion** > **Rasterize (Vector to Raster)**.

*   **Exercise 10:** Rasterize country boundaries to create a spatial analysis mask aligning with the DEM grid.
    
    1. Load `ne_10m_admin_0_countries` and select Nepal.
    
    2. Open **Raster** > **Conversion** > **Rasterize (Vector to Raster)**.
    
    3. Set the parameters:
       
       * **Input Layer:** `ne_10m_admin_0_countries` (check **Selected features only**).
       
       * **Field to use for burn-in value:** Select `MAPCOLOR7` (or enter a fixed value of `1` under **A fixed value to write**).
       
       * **Output raster size units:** Select **Pixels**.
       
       * **Width/Horizontal resolution:** Match the width of `output_hh.tif` (e.g. use the `output_hh.tif` layer's properties to check pixel dimensions, or set resolution to match).
       
       * **Output bounds:** Click `...` > **Use Extent from** > Select `output_hh.tif` (forces the grid extent to match the DEM).
       
       * **Rasterized:** Save as `nepal_raster_mask.tif`.
    
    4. Click **Run**. This generates a grid matching the DEM cell structure, containing $1$ over Nepal and $0$ (or NoData) elsewhere.

---

## 8. Interactive Classroom Discussion Scenarios

### Scenario A: Mismatched Grid Alignment in Raster Calculator

A hydrologist runs the Raster Calculator to subtract two rasters: `"dem_2020@1" - "dem_2010@1"`. The calculation completes successfully, but the output file is entirely empty (all values are NoData). Upon inspection, they see that:

*   `dem_2020` has a cell size of $30\text{m}$ in UTM CRS.

*   `dem_2010` has a cell size of $0.000277^\circ$ in WGS84 geographic CRS.

Explain why the output is blank and list the exact preprocessing steps required to resolve the issue.

??? check "Answer Key - Grid Alignment"

    1. **The Root Cause:** QGIS's Raster Calculator processes rasters cell-by-cell matching exact coordinates. When coordinate systems differ (UTM meters vs. WGS84 degrees), cells do not overlap in spatial coordinates, and cell boundaries do not align. A cell calculation with any NoData or non-overlapping region defaults to NoData.
    
    2. **Resolution Workflow:**
       
       * **Reproject:** Run **Warp (Reproject)** on `dem_2010` to convert it to the UTM CRS of `dem_2020`.
       
       * **Resample & Align:** During reprojection, specify the output cell size to match the target ($30\text{m}$) and use the **Bilinear** resampling method.
       
       * **Set Extent:** Set the output bounds of the warp to match the extent of `dem_2020`.
       
       * **Run Calculator:** Run the subtraction query on the reprojected, aligned layers.

### Scenario B: Choosing the Right Resampling Method

An agricultural GIS specialist is working with a land cover classification map (pixel values representing Forest = 1, Agriculture = 2, Urban = 3). They need to reproject this dataset to match a projected watershed boundary map. During reprojection, they leave the default resampling method set to **Bilinear**. 

Describe the problem this will create in the output land cover map and recommend the correct settings.

??? check "Answer Key - Resampling Categorical Data"

    1. **The Problem:** Bilinear interpolation calculates a weighted average of the four nearest pixels. If a boundary pixel lies between Forest ($1$) and Urban ($3$), Bilinear interpolation calculates $2$ (Agriculture). This introduces completely false land cover classifications that did not exist in the original data.
    
    2. **The Correct Setting:** For categorical/thematic data, the **Nearest Neighbor** resampling method must be selected. This method copies the exact class value of the nearest cell without performing any mathematical interpolation, preserving the integrity of the integer categories.
