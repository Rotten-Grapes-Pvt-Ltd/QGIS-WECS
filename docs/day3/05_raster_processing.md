# Raster Processing and Map Algebra

Rasters represent continuous geographic surfaces (such as elevation, temperature, or spectral indices) using a grid of cells (pixels). Unlike vector shapes, raster processing is performed on a cell-by-cell basis across grid rows and columns. This section covers map algebra, raster warping/resampling, reclassification, and raster-vector conversions.

---

## 1. Raster Structure and Cell Concepts

A raster grid consists of columns and rows defined by spatial coordinate parameters:

```text
    RASTER CELL LAYOUT
    +-------+-------+-------+
    | 245.8 | 246.2 | 246.5 |  <-- Cells containing values (e.g., elevation in meters)
    +-------+-------+-------+
    | 244.1 | 244.9 | 245.2 |  
    +-------+-------+-------+
    | 243.0 | -9999 | 243.8 |  <-- -9999 represents NoData (null) cells
    +-------+-------+-------+
    [Cell Size: 30m x 30m]
```

* **Cell Size (Resolution):** The geographic width and height of a single pixel (e.g., $10\text{ m}$ for Sentinel-2, $30\text{ m}$ for Landsat/SRTM). All calculations inside a cell are averaged across this area.

* **NoData Value:** A designated placeholder number (e.g., `-9999` or `NaN`) indicating the absence of data. Operations on a NoData cell always output NoData.

* **Bands:** A single raster file can contain multiple stacked layers (bands). For example, Sentinel-2 images contain 13 separate spectral bands.

---

## 2. Map Algebra and the Raster Calculator

The **Raster Calculator** (**Raster** > **Raster Calculator**) evaluates mathematical or logical expressions on one or more raster grids on a cell-by-cell basis:

$$\text{Output Cell} = f(\text{Input Cell}_A, \text{Input Cell}_B)$$

### Common Operations

* **Difference Grids (Sedimentation/Erosion):** Subtracting an older DEM from a newer DEM to map structural volume changes over time:
  `"DEM_2024@1" - "DEM_2014@1"`

* **Conditional Logic (Binary Masking):** Extracting areas above $3000\text{ meters}$ elevation. The output pixel is assigned $1$ if true, and $0$ if false:
  `"DEM@1" > 3000`

* **Multi-Criteria Query:** Extracting high-risk zones where elevation is above $2000\text{ m}$ and slope is greater than $25^{\circ}$:
  `("DEM@1" > 2000) * ("Slope@1" > 25)`

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

---

## 4. Raster Reclassification

Reclassification groups range-based cell values into simplified, discrete classifications (converting continuous rasters to categorical rasters).

* **Accessing:** Open the Processing Toolbox and select **Raster Analysis** > **Reclassify by Table**.

* **Scenario:** Classifying topographic slope grids into flood velocity hazard zones:

```text
    SLOPE VALUE RANGE       CLASS VALUE     THEMATIC CATEGORY
    0.0 to  2.0 degrees -->     1       --> Flat (High Flood Risk)
    2.0 to 15.0 degrees -->     2       --> Moderate (Medium Risk)
    15.0 to 90.0 degrees -->    3       --> Steep (Low Flood Risk)
```

---

## 5. Raster-Vector Conversions

In spatial analysis workflows, you often need to transfer data between vector formats and raster structures:

### Raster to Vector (Polygonize)

Converts a raster grid into vector polygons. Adjacent pixels with identical integer values are merged into single polygon boundaries.

* **Accessing:** Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)**.

* **Hydrological Application:** Converting a binary water classification raster (where $1 = \text{water}$) into vector polygons to calculate perimeter shapes and export boundaries.

### Vector to Raster (Rasterize)

Converts vector geometries (points, lines, or polygons) into a grid of raster cells.

* **Accessing:** Go to **Raster** > **Conversion** > **Rasterize (Vector to Raster)**.

* **Hydrological Application:** Converting sub-catchment boundary polygons into a raster mask grid to run joint cell math inside the Raster Calculator.
