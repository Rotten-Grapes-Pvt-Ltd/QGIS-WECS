# Practical Laboratory and Mini-Assignment

This session combines a guided step-by-step laboratory tutorial (Part A) with an independent mini-assignment (Part B). You will apply the terrain processing and hydrological routing skills learned throughout Module 4.

---

## Part A: Guided Laboratory Workflow

This tutorial guides you through setting up a project workspace, conditioning a raw Digital Elevation Model, routing water, extracting stream channels, and delineating a watershed boundary.

### 1. Workspace Configuration
Before starting calculations, configure your QGIS project:

1.  Create a standard directory tree in your local workspace:
    *   `data/raw/` (for raw inputs).
    *   `data/processed/` (for intermediate and final outputs).

2.  Open QGIS. Save a new project as `Day4_Hydrology_Analysis.qgz`.

3.  Set the project CRS to **WGS 84 / UTM Zone 45N (EPSG:32645)**.

4.  Copy the local DEM `output_hh.tif` from the [docs/data/Natural_Earth_quick_start/DEM/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/) folder into your local `data/raw/` folder and load it into QGIS.

### 2. DEM Reprojection & Conditioning
Ensure the DEM coordinates are in meters and remove sinks to establish continuous flow:

1.  **Reproject DEM:** Go to **Raster** > **Projections** > **Warp (Reproject)...**. Set Input to `output_hh.tif`, Target CRS to `EPSG:32645`, Resampling to **Bilinear**, and save as `data/processed/output_hh_utm.tif`.

2.  **Fill Sinks:** Go to **Processing Toolbox** > **WhiteboxTools** > **Hydrological Analysis** > **Fill Depressions** (or **GRASS** > **Raster (r.*)** > **r.fill.dir**).
    *   **DEM:** `data/processed/output_hh_utm.tif`.
    *   **Filled DEM:** Save as `data/processed/filled_dem.tif`.
    *   Click **Run**.

### 3. Flow Direction and Accumulation
Compute flow direction paths and cumulative upstream drainage areas:

1.  Navigate to **Processing Toolbox** > **WhiteboxTools** > **Hydrological Analysis** > **D8 Flow Accumulation** (or **GRASS** > **Raster (r.*)** > **r.watershed**).

2.  Configure parameters:
    *   **Elevation:** `data/processed/filled_dem.tif`.
    *   **Flow Accumulation:** Save as `data/processed/flow_accumulation.tif`.

3.  Click **Run**. Style `flow_accumulation.tif` using a logarithmic color scale to visualize stream paths.

### 4. Stream Network Extraction
Isolate cells representing major streams and convert them into vector lines:

1.  Open **Raster** > **Raster Calculator...**.

2.  Apply an extraction threshold of 1000 cells (delineating pixels with at least $1000$ pixels contributing drainage):
    `"flow_accumulation@1" >= 1000`

3.  Save output as `data/processed/stream_network_binary.tif`. Click **OK**.

4.  Vectorize the stream grid: Navigate to **Processing Toolbox** > **GDAL** > **Raster Conversion** > **Polygonize (Raster to Vector)...** (or use **WhiteboxTools** > **Stream Network Analysis** > **Raster Streams To Vector**).
    *   **Grid:** `stream_network_binary.tif`.
    *   **Class Selection:** Set to **each class** or filter for class `1`.
    *   Save output as a vector layer `data/processed/vector_streams.gpkg`.

### 5. Catchment Delineation
Delineate the watershed boundary draining to a selected outlet coordinate:

1.  Navigate to **Processing Toolbox** > **WhiteboxTools** > **Hydrological Analysis** > **Watershed** (or **GRASS** > **Raster (r.*)** > **r.water.outlet**).

2.  Configure parameters:
    *   **Elevation / Flow Direction:** Select the flow direction grid.
    *   **Target X / Target Y:** Click the map tool button (`...`) and click a point on your stream network representing the basin outlet.
    *   **Upslope Area:** Save output as a raster `data/processed/basin_boundary_raster.tif`.

3.  Click **Run**. Convert this to a vector layer: Go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)...**. Select `basin_boundary_raster.tif` and save as a vector polygon `data/processed/basin_boundary_polygon.gpkg`.

4.  Style the polygon with a transparent fill and a thick black outline.

### 6. Soil Erosion Susceptibility (RUSLE)
Calculate the topographic LS factor and compile the annual soil loss grid:

1.  **Calculate LS Factor:** Navigate to **Processing Toolbox** > **WhiteboxTools** > **Hydrological Analysis** > **Lsf Factor** (or **GRASS** > **Raster (r.*)** > **r.uslek**).
    *   **Elevation:** `data/processed/filled_dem.tif`.
    *   **Flow Accumulation:** `data/processed/flow_accumulation.tif`.
    *   **LS Factor:** Save output as `data/processed/ls_factor.tif`.
    *   Click **Run**.

2.  **Compile Final Soil Loss:** Open **Raster** > **Raster Calculator...**.
    *   Multiply the local factor rasters ($R, K, C, P$, assuming default constants if local grids are unavailable, e.g. $R = 250$, $K = 0.25$, $C = 0.1$, $P = 1.0$):
        `250 * 0.25 * "ls_factor@1" * 0.1 * 1.0`
    *   Save output as `data/processed/annual_soil_loss.tif`. Click **OK**.
    *   Style `annual_soil_loss.tif` using **Singleband pseudocolor** classified into 5 categories ($0-5$ Slight, $5-10$ Moderate, $10-20$ High, $20-40$ Very High, $>40$ Severe).

### 7. Rainfall Interpolation and Catchment Zonal Statistics
Interpolate point weather station records and compute volumetric catchment rainfall:

1.  **IDW Interpolation:** Navigate to **Processing Toolbox** > **QGIS** > **Raster Analysis** > **IDW Interpolation**.
    *   **Vector Layer:** Select a point gauge layer (e.g. `rain_gauges.shp`).
    *   **Interpolation Attribute:** Select the rainfall column (e.g. `precip_mm`).
    *   **Extent:** Select **Use Extent from** > `data/processed/basin_boundary_polygon.gpkg`.
    *   **Pixel Size:** Set to `30` meters.
    *   **Interpolated:** Save output as `data/processed/rainfall_idw.tif`. Click **Run**.

2.  **Calculate Zonal Statistics:** Go to **Processing Toolbox** > **Raster Analysis** > **Zonal Statistics**.
    *   **Input Raster:** `data/processed/rainfall_idw.tif`.
    *   **Vector Layer Containing Zones:** `data/processed/basin_boundary_polygon.gpkg`.
    *   **Output Column Prefix:** Type `rain_`.
    *   **Statistics to Calculate:** Check **Mean** and **Sum**.
    *   Click **Run**.

3.  **Compute Volume in Field Calculator:** Open the attribute table of your catchment polygon. Open the **Field Calculator**, create a new decimal field `precip_m3`, and enter the volumetric equation:
    `("rain_mean" / 1000) * $area`
    Click **OK**. This calculates the total cubic meters ($m^3$) of rainfall entering the basin.

### 8. Height Above Nearest Drainage (HAND) Mapping
Delineate relative topography above stream channels to locate low-lying flood inundation hazard zones:

1.  Navigate to **Processing Toolbox** > **WhiteboxTools** > **Hydrological Analysis** > **Elevation Above Stream** (or use **r.stream.distance** in GRASS).
    *   **Elevation:** `data/processed/filled_dem.tif`.
    *   **Streams:** `data/processed/stream_network_binary.tif`.
    *   **Relative Heights:** Save output as `data/processed/hand_model.tif`.
    *   Click **Run**.

2.  Style `hand_model.tif` in the **Layer Styling Panel** using a classified color ramp. Highlight cells between $0\text{ m}$ and $2\text{ m}$ in red to map regions susceptible to local flood inundation.

### 9. Reservoir Stage-Volume Capacity Curve Analysis
Calculate reservoir storage capacities and surface areas at varying water levels:

1.  **Clip DEM to Reservoir Catchment:** Delineate the upslope catchment draining through a planned dam axis. Use the **Raster Calculator** to clip the DEM to this catchment boundary. Save as `data/processed/reservoir_dem.tif`.

2.  **Calculate Volume at Target Stage:** Go to **Processing Toolbox** > **QGIS Raster Analysis** > **Raster Surface Volume**.
    *   **Input Layer:** `data/processed/reservoir_dem.tif`.
    *   **Base Threshold:** Enter the target pool height elevation (e.g. `480` meters).
    *   **Method:** Select **Count Only Below**.
    *   **Volume / Area Output:** Save as a table `data/processed/stage_volume_480.html`.
    *   Click **Run**. Open the HTML report to extract pool area ($m^2$) and storage volume ($m^3$).

---

## Part B: Mini-Assignment: Watershed Terrain Characterization

### 1. Objective
You will independently delineate a specific sub-watershed, calculate its topographic statistics, and compile a publication-quality map layout.

### 2. Required Tasks
Using the layers created in Part A, execute the following steps:

1.  **Delineate a Sub-basin:**
    *   Identify a downstream confluence point on `vector_streams.gpkg`.
    *   Run GRASS **r.water.outlet** or WBT **Watershed** using that coordinate to delineate the upstream catchment boundary.
    *   Convert the output raster to a vector polygon layer named `catchment_boundary.gpkg`.

2.  **Calculate Topographic Statistics (Zonal Statistics):**
    *   Run **Zonal Statistics** using `catchment_boundary.gpkg` as the zone and `data/processed/output_hh_utm.tif` as the input raster. Compute the **Mean**, **Min**, and **Max** elevations.
    *   Generate a **Slope** grid (`slope_degrees.tif`) via GDAL Slope. Run **Zonal Statistics** using the slope grid to find the catchment's **Mean Slope** in degrees (`slope_mean`).
    *   Open the attribute table of `catchment_boundary.gpkg` and calculate the total catchment surface area in square kilometers: `$area / 1000000`.

3.  **Generate Stream Ordering:**
    *   Open WhiteboxTools **Strahler Stream Order**. Select the filled DEM and your stream network. Set Method to **Strahler** and output `strahler_order.tif`.
    *   Run the QGIS **Polygonize (Raster to Vector)** tool to convert the Strahler raster to vector lines named `ordered_streams.gpkg`.

4.  **Compose Print Layout Map:**
    *   Create an A4 landscape Print Layout in QGIS.
    *   Render the styled layers in the map frame:
        *   `ordered_streams.gpkg` styled graduated by the `ORDER` field (tributaries at $0.2\text{ mm}$, main channels at $1.5\text{ mm}$).
        *   `catchment_boundary.gpkg` styled with an empty fill and a solid $0.8\text{ mm}$ red border.
        *   **Slope** raster styled semi-transparent pseudocolor overlaying a **Hillshade** layer.
    *   Add a text block table detailing:
        *   **Total Drainage Area** ($km^2$).
        *   **Elevation Range** ($meters$).
        *   **Mean Slope** ($degrees$).
    *   Include essential cartographic elements: Title, North Arrow, scale bar, legend, and UTM coordinate grid.

### 3. Submission Files
Export your print layout to PDF and compile a ZIP file containing:

1.  `Basin_Terrain_Characterization_Map.pdf`.

2.  `Watershed_Terrain_Data.gpkg` (containing `catchment_boundary` and `ordered_streams`).

3.  Your QGIS project file saved with Relative Paths.
