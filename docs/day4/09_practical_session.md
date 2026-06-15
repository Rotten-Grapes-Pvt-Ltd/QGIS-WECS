# Practical Session: Watershed Delineation in QGIS

This practical session walks you through using the SAGA tools inside QGIS to process a DEM, calculate flow directions, extract stream networks, and delineate a watershed boundary.

---

## 1. DEM Processing and Sink Filling

1. Load `raw_elevation.tif` in QGIS.

2. Go to the **Processing Toolbox** and search for **SAGA** > **Terrain Analysis - Preprocessing** > **Fill Sinks (Wang & Liu)**.

   * **DEM:** `raw_elevation.tif`.

   * **Filled DEM:** Save as `filled_dem.tif`.

   * Click **Run**. Sinks are removed, ensuring continuous flow.

---

## 2. Flow Direction and Accumulation

1. Search the Processing Toolbox for **SAGA** > **Terrain Analysis - Hydrology** > **Catchment Area (Parallel)**.

   * **Elevation:** `filled_dem.tif`.

   * **Flow Direction:** Save as `flow_direction.tif`.

   * **Catchment Area (Accumulation):** Save as `flow_accumulation.tif`.

   * Click **Run**.

2. Examine `flow_accumulation.tif`. Pixels with high values represent stream channels.

---

## 3. Stream Network Extraction

1. Open the **Raster Calculator**.

2. Build an expression to extract streams by setting a threshold (e.g., cell accumulation $> 5000$):
   `"flow_accumulation@1" > 5000`

3. Save the output raster as `stream_network.tif` (contains $1	ext{s}$ along stream lines, and $0	ext{s}$ elsewhere).

4. Convert the stream raster to vector lines using **SAGA** > **Terrain Analysis - Channels** > **Vectorising Grid Classes**.

---

## 4. Catchment Delineation

1. Search the Processing Toolbox for **SAGA** > **Terrain Analysis - Hydrology** > **Upslope Area**.

   * **Elevation:** `filled_dem.tif`.

   * **Target Cell X/Y:** Click the coordinates of the watershed outlet on the map canvas.

   * Save the output as `watershed_boundary.gpkg` (converts upslope area to a vector polygon).
