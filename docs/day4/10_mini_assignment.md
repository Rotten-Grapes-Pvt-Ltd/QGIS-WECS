# Mini-Assignment: Watershed Terrain Characterization

In this assignment, you will apply the terrain analysis and flow routing skills learned during Day 4 to preprocess a digital elevation model, delineate a watershed, extract its stream network, compute Strahler stream orders, calculate catchment statistics, and compile a landscape map layout. This assignment requires approximately 1 hour to complete.

---

## 1. Objective

Condition a raw elevation dataset, delineate the catchment boundary upstream of a designated pour point coordinate, perform Strahler stream ordering on the drainage network, calculate catchment-wide elevation and slope statistics, and export a cartographically sound print layout map.

---

## 2. Provided Datasets

The following raw layers are located in your project directory:

* `/data/raster/raw_dem.tif` (Raw elevation model, WGS 84 / UTM Zone 45N / EPSG:32645).

* `/data/vector/outlet_point.gpkg` (A point vector layer marking the basin outlet coordinates).

---

## 3. Step-by-Step Requirements

### Step 1: DEM Conditioning and Preprocessing

1. Open QGIS and load the `raw_dem.tif` raster.

2. Run SAGA's **Fill Sinks (Wang & Liu)** tool to remove depressions. Save the output as `conditioned_dem.tif` inside your processed data directory.

---

### Step 2: Flow Routing and Basin Delineation

1. Run SAGA's **Catchment Area (Parallel)** tool on `conditioned_dem.tif` to generate:

   * `flow_direction.tif` (D8 routing direction grid).

   * `flow_accumulation.tif` (Upstream contributing cell grid).

2. Delineate the upstream contributing watershed:

   * Run SAGA's **Upslope Area** tool using `flow_direction.tif`.

   * **Target X / Target Y:** Select the coordinate of the outlet marker point in `outlet_point.gpkg`.

   * Save the output raster as `basin_boundary_raw.tif`.

3. Convert `basin_boundary_raw.tif` into a vector polygon using **Polygonize (Raster to Vector)**. Save it in your project database as `catchment_boundary`.

---

### Step 3: Stream Network and Strahler Stream Ordering

Identify the stream network and classify the channel hierarchy:

1. Run SAGA's **Channel Network** or **Stream Order** tool inside the Processing Toolbox.

   * **Elevation:** `conditioned_dem.tif`.

   * **Flow Direction:** `flow_direction.tif`.

   * **Method:** Select **Strahler** ordering.

   * **Threshold:** Set to $5000\text{ cells}$.

2. Save the output vector lines in your database as `ordered_streams`. The attribute table will contain a column named `ORDER` (ranging from $1$ to $5$ depending on complexity).

---

### Step 4: Extracting Catchment Statistics

Calculate the structural metrics of the newly delineated catchment:

1. Run **Zonal Statistics** on your catchment boundary:

   * **Raster Layer:** `conditioned_dem.tif`.

   * **Vector Layer:** `catchment_boundary`.

   * Calculate `Mean`, `Min`, and `Max` elevation.

2. Run **Zonal Statistics** a second time using the terrain **Slope** raster to extract the catchment's average slope in degrees (`slope_mean`).

---

### Step 5: Map Layout Composition

Compile an A4 landscape print layout:

1. Add a map frame rendering the following styled layers:

   * **Top Layer:** `ordered_streams` styled using **Categorized Symbology** based on the stream `ORDER` field. Apply a sequential blue palette where line thickness increases with stream order (e.g., Order 1 $= 0.15\text{ mm}$, Order 2 $= 0.30\text{ mm}$, Order 3 $= 0.50\text{ mm}$, Order 4 $= 0.80\text{ mm}$).

   * **Middle Layer:** `catchment_boundary` styled with an empty fill and a solid $0.7\text{ mm}$ red border.

   * **Bottom Layer:** Styled **Slope** raster (semi-transparent pseudocolor) overlaying a **Hillshade** layer.

2. Add a text block table detailing the following calculated catchment statistics:

   * **Total Drainage Area:** (calculated via `$area / 1000000` in $km^2$).

   * **Elevation Range:** Min and Max elevation ($meters$).

   * **Mean Slope:** Average slope angle ($degrees$).

3. Add standard cartographic elements: Title, North Arrow, scale bar, legend (renamed with clean labels), and UTM coordinate graticules.

---

## 4. Submission Instructions

Export your layout map as a PDF. Compile a ZIP archive named `Day4_Assignment_[YourName].zip` containing:

1. Your exported PDF map (`Basin_Terrain_Characterization_Map.pdf`).

2. The completed GeoPackage database container (`Watershed_Terrain_Data.gpkg`) containing the `catchment_boundary` and `ordered_streams` vector layers.

3. Your QGIS project file (`Day4_Assignment.qgz`) saved with relative paths.
