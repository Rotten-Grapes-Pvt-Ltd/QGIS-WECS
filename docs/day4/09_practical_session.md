# Practical Session: Watershed Delineation and Stream Network Extraction

This practical session walks you through the step-by-step workflow of conditioning a digital elevation model, routing water, extracting stream channels using threshold calculations, and delineating a watershed boundary upstream of a specific outlet.

---

## 1. Setting Up the Workspace

Before starting your calculations, organize your directories and configure your QGIS project:

1. Create the following local directory structure:

   * `data/raw/` (Contains your raw input DEM: `raw_elevation.tif`).

   * `data/processed/` (Where all intermediate and final outputs will be saved).

2. Open QGIS and create a new project. Save it as `Day4_Hydrology_Analysis.qgz`.

3. Set the project Coordinate Reference System (CRS) to **WGS 84 / UTM Zone 45N (EPSG:32645)**.

4. Import `raw_elevation.tif` to the Map Canvas.

---

## 2. DEM Preprocessing and Sink Filling

Remove artificial depressions from the DEM to ensure continuous water flow routing:

1. Open the **Processing Toolbox** (`Ctrl+Alt+T`).

2. Navigate to **SAGA** > **Terrain Analysis - Preprocessing** > **Fill Sinks (Wang & Liu)**.

   * **DEM:** `raw_elevation.tif`

   * **Filled DEM:** Click `...` > **Save to File**. Save as `data/processed/filled_dem.tif`.

   * *Note: Uncheck "Minimum Slope" and "Filled Basins" options if you only need the filled DEM output.*

3. Click **Run**. The filled DEM will load in the Layers Panel. Compare the filled DEM and the raw DEM; the artificial sinks have been filled to allow continuous flow.

---

## 3. Flow Direction and Accumulation Calculations

Compute downhill flow directions and compile the cumulative upstream contributing area:

1. In the Processing Toolbox, search for SAGA's **Catchment Area (Parallel)** tool (**SAGA** > **Terrain Analysis - Hydrology** > **Catchment Area (Parallel)**).

2. Configure the tool parameters:

   * **Elevation:** `filled_dem.tif`

   * **Method:** Select **Deterministic 8 (D8)** (standard single flow routing).

   * **Flow Direction:** Save as `data/processed/flow_direction.tif`.

   * **Catchment Area:** Save as `data/processed/flow_accumulation.tif`.

3. Click **Run**. 

4. **Visual Inspection:** Turn off all layers except `flow_accumulation.tif`. Apply a logarithmic color ramp or adjust the layer styling min/max values to see the linear stream channels (cells with high accumulation values) stand out clearly.

---

## 4. Extracting the Stream Network

Isolate cells that represent major streams and convert them into vector lines:

1. Open the **Raster Calculator** (**Raster** > **Raster Calculator**).

2. We will apply an extraction threshold of $5000\text{ contributing cells}$ (meaning a pixel must have at least $5000$ pixels draining through it to be classified as a stream). Write the logical expression:
   `"flow_accumulation@1" > 5000`

3. **Output Layer:** Save as `data/processed/stream_network_binary.tif`. Click **OK**. The output is a binary raster containing $1\text{s}$ (the streams) and $0\text{s}$ (non-stream cells).

4. Convert this binary raster into vector lines:

   * Search the Processing Toolbox for SAGA's **Vectorising Grid Classes** (**SAGA** > **Vector <-> Raster** > **Vectorising Grid Classes**).

   * **Grid:** `stream_network_binary.tif`

   * **Class Selection:** Set to **each class** or filter for class value `1`.

   * Save the vector output to your GeoPackage database as `vector_streams`.

---

## 5. Delineating the Catchment Boundary (Upslope Area)

Delineate the watershed boundary draining to a planned outlet coordinate:

1. Search for SAGA's **Upslope Area** tool inside the Processing Toolbox.

2. Configure the parameters:

   * **Elevation:** `filled_dem.tif`

   * **Method:** Select **Deterministic 8 (D8)**.

   * **Target X / Target Y:** Click the map selector button (`...`) and click a point on your vector stream network representing your chosen basin outlet (pour point).

   * **Upslope Area:** Save the output as a raster `data/processed/basin_boundary_raster.tif`.

3. Click **Run**. The resulting raster will isolate all upstream pixels that drain into your selected coordinate.

4. Convert the raster boundary into a vector polygon:

   * Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)**.

   * **Input Layer:** `basin_boundary_raster.tif`

   * Save the output vector polygon in your database as `basin_boundary_polygon`.

5. Style the final polygon layer with an empty fill and a thick black outline. This represents your delineated catchment.
