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

* **Cell Size (Resolution):** The geographic width and height of a single pixel (e.g., $30\text{ m}$ for the ALOS PALSAR / SRTM grids). All calculations inside a cell are averaged across this area.
* **NoData Value:** A designated placeholder number (e.g., `-9999` or `-3.402823e+38`) indicating the absence of data. Operations on a NoData cell always output NoData.
* **Bands:** A single raster file can contain multiple stacked layers (bands). For example, satellite scenes contain separate spectral bands (Red, Green, Blue, Near-Infrared), whereas elevation DEMs are singleband rasters.

---

## 2. Map Algebra and the Raster Calculator

The **Raster Calculator** (**Raster** > **Raster Calculator**) evaluates mathematical or logical expressions on one or more raster grids on a cell-by-cell basis:

$$\text{Output Cell} = f(\text{Input Cell}_A, \text{Input Cell}_B)$$

### Exercise 1: Extract High-Altitude Zones (Binary Masking)
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

### Exercise 2: Extract Altitudinal Valleys (Multi-Criteria Masking)
Extract middle-altitude valleys and hills ranging strictly between $1500$ and $3500\text{ meters}$.

1. Open the **Raster Calculator**.
2. Enter the following logical combination expression:
   ```sql
   ("output_hh@1" >= 1500) * ("output_hh@1" <= 3500)
   ```
   *(Multiplying these boolean statements acts as a logical AND operation—only cells where both conditions are true ($1 \times 1$) will output $1$.)*
3. Save the output as `valley_hills_mask.tif` and click **OK**.

---

## 3. Raster Warping and Resampling Algorithms

When running analysis combining multiple rasters, they must share the exact same cell sizes, extents, and spatial alignments.

* **Raster Warping (Reprojection):** Changes the Coordinate Reference System (CRS) of a raster grid. Access via **Raster** > **Projections** > **Warp (Reproject)**.
* **Resampling:** Injected during reprojection or cell resizing to calculate new pixel values from the old grid layout. Selecting the wrong resampling algorithm will corrupt your data:

| Resampling Method | Ideal Data Type | Calculation Logic | Hydrological Impact |
| :--- | :--- | :--- | :--- |
| **Nearest Neighbor** | Categorical / Discrete | Copies the value of the nearest cell. | Preserves class values (e.g., landuse codes $1, 2, 3$). Using it on continuous elevation data creates blocky steps. |
| **Bilinear** | Continuous | Calculates a linear average of the 4 nearest pixels. | Smooths values. Ideal for continuous rasters like elevation grids (DEMs) or temperature surfaces. |
| **Cubic Convolution** | Continuous | Fits a smooth curve across the 16 nearest pixels. | High precision, but computationally heavy. Best for high-resolution imagery and ortho-rectification. |

### Exercise: Reproject and Resample the DEM
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

## 4. Raster Reclassification

Reclassification groups continuous cell values into simplified, discrete categories (converting continuous rasters to categorical rasters).

* **Accessing:** Open the Processing Toolbox and select **Raster Analysis** > **Reclassify by Table**.

### Exercise: Reclassify DEM into Altitudinal Zones
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

## 5. Raster-Vector Conversions

In spatial analysis workflows, you often need to transfer data between vector formats and raster structures:

### Raster to Vector (Polygonize)

Converts a raster grid into vector polygons. Adjacent pixels with identical integer values are merged into single polygon boundaries.

* **Accessing:** Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)**.
* **Exercise:** Convert the binary high-altitude raster mask created in Section 2 into vector polygons.
  1. Open **Raster** > **Conversion** > **Polygonize**.
  2. **Input Layer:** Select `high_altitude_mask.tif` (the binary mask).
  3. **Name of the field to create:** Enter `altitude_class` (stores values $1$ or $0$).
  4. **Vectorized:** Save as a GeoPackage table named `high_altitude_peaks`.
  5. Click **Run**. Select the polygons where `altitude_class = 1` to isolate the peaks.

### Vector to Raster (Rasterize)

Converts vector geometries (points, lines, or polygons) into a grid of raster cells.

* **Accessing:** Go to **Raster** > **Conversion** > **Rasterize (Vector to Raster)**.
* **Exercise:** Rasterize country boundaries to create a spatial analysis mask aligning with the DEM grid.
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
