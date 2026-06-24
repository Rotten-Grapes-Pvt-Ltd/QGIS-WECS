# Watershed Characterization

Watershed characterization involves extracting the topographic and drainage properties of a catchment area from a digital elevation model (DEM). These physical properties govern how precipitation is routed across land surfaces and eventually discharged at the basin outlet.

---

## 1. Core Objectives

*   **Delineate** boundary divides that define the surface runoff drainage area.

*   **Extract** drainage networks (streams and rivers) and classify them by order.

*   **Compute** quantitative morphometric parameters that describe the geometry, shape, and relief of the watershed.

---

## 2. Key GIS Inputs

*   **Digital Elevation Model (DEM):** High-resolution elevation data (such as Copernicus DEM 30m, SRTM 30m, or ALOS AW3D30).

*   **Pour Point Coordinates:** The coordinates of the outlet point (e.g., a stream gauging station, reservoir inlet, or river confluence) where runoff exits the watershed.

---

## 3. Step-by-Step Delineation Workflow

In desktop GIS (QGIS with SAGA or WhiteboxTools), catchment delineation follows a sequential raster processing chain where the output of each geoprocessing tool becomes the mandatory input for the next:

```text
    +-----------+     +-----------------+     +-----------------+
    |  Raw DEM  | --> | Fill Sinks/Pits | --> | Flow Direction  |
    +-----------+     +-----------------+     +-----------------+
                                                       |
                                                       v
    +-----------+     +-----------------+     +-----------------+
    | Watershed | <-- | Stream Network  | <-- |Flow Accumulation|
    | Boundary  |     |   Extraction    |     |                 |
    +-----------+     +-----------------+     +-----------------+
```

1.  **Fill Sinks / Pits:**
    
    *   **What We Are Doing:** Filling the natural and artificial depressions (sinks or pits) in the raw elevation dataset to create a hydrologically continuous terrain surface.
    
    *   **Why This Step is Needed:** Elevation datasets derived from remote sensing contain radar speckle, forest canopy blockages, and integer rounding errors. Sinks act as digital water traps. If left unconditioned, the flow routing engine cannot assign flow directions to these cells, resulting in broken stream networks and incomplete catchment boundaries.
    
    *   *Input:* Raw DEM raster (`output_hh_utm.tif`).
    
    *   *Output:* Conditioned, depressionless elevation grid (`filled_dem.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Fill Sinks (Wang & Liu)**.
        
        *   **DEM:** Select `output_hh_utm.tif`.
        
        *   **Minimum Slope:** Set to `0.01` (to force a micro-slope downstream across filled flat shelves).
        
        *   **Filled DEM:** Click the file dialog button and save as `filled_dem.tif`. (Uncheck other outputs like Flow Directions).

2.  **Flow Direction:**
    
    *   **What We Are Doing:** Calculating the direction of steepest downhill slope for every cell in the elevation model.
    
    *   **Why This Step is Needed:** The D8 algorithm determines which of the eight surrounding cells has the steepest downward slope. This flow direction map acts as the routing blueprint for water moving across the watershed. Without it, the GIS engine cannot calculate downstream flow accumulation or trace catchment boundaries.
    
    *   *Input:* Conditioned DEM (`filled_dem.tif`) from **Step 1**.
    
    *   *Output:* Flow Direction grid (`flow_direction.tif`), encoding flow paths.
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
        
        *   **Elevation:** Select `filled_dem.tif` (the output from Step 1).
        
        *   **Method:** Select `[0] Deterministic 8 (D8)`.
        
        *   **Flow Directions:** Save as `flow_direction.tif`. (Uncheck other outputs).

3.  **Flow Accumulation:**
    
    *   **What We Are Doing:** Calculating the cumulative number of upstream cells that drain into each downhill pixel.
    
    *   **Why This Step is Needed:** Flow accumulation simulates runoff concentration. Cells with high accumulation counts have large drainage areas feeding into them, which locates stream channels and river valleys. Ridge divides and mountaintops will have an accumulation value of zero, as no upstream areas drain into them.
    
    *   *Input:* Conditioned DEM (`filled_dem.tif` from **Step 1**).
    
    *   *Output:* Contributing drainage area grid (`flow_accumulation.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
        
        *   **Elevation:** Select `filled_dem.tif`.
        
        *   **Method:** Select `[0] Deterministic 8 (D8)`.
        
        *   **Flow Accumulation:** Save as `flow_accumulation.tif`. (Uncheck other outputs).

4.  **Stream Network Extraction:**
    
    *   **What We Are Doing:** Extracting stream segments by filtering out cells that do not meet a minimum contributing drainage area.
    
    *   **Why This Step is Needed:** Continuous flow accumulation cells contain values ranging from zero to millions. To delineate stream paths, we apply a threshold to isolate pixels representing significant drainage concentration. This converts the continuous contributing area map into a binary stream/non-stream raster.
    
    *   *Input:* Flow Accumulation grid (`flow_accumulation.tif`) from **Step 3**.
    
    *   *Output:* Binary stream channel grid (`stream_network_binary.tif` where stream = 1, land = 0).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Channels** > **Channel Subnetwork**.
        
        *   **Elevation:** Select `filled_dem.tif`.
        
        *   **Initiation Threshold:** Set to `1000` (meaning streams will start where flow accumulation $\ge 1000$ pixels).
        
        *   **Channel Network:** Save as `stream_network_binary.tif`.
        
        *   *Alternative (QGIS Native Raster Calculator):* Write `"flow_accumulation@1" >= 1000` and save the result as `stream_network_binary.tif`.

5.  **Watershed Delineation:**
    
    *   **What We Are Doing:** Tracing all upstream pixels that drain into a specific outlet (pour point).
    
    *   **Why This Step is Needed:** Hydrologists must calculate water budgets, peak runoffs, and sediment loads for specific drainage areas. Watershed delineation outlines the catchment boundary uphill of a chosen outlet (e.g. a river gauge station or dam site), separating the contributing runoff area from the surrounding basins.
    
    *   *Input:* Conditioned DEM (`filled_dem.tif` from **Step 1**).
    
    *   *Output:* Catchment boundary grid (`watershed_basin.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Upslope Area**.
        
        *   **Elevation:** Select `filled_dem.tif`.
        
        *   **Target X Coordinate / Target Y Coordinate:** Click the coordinate picker button, then click on the active stream centerline on the map where your outlet sits.
        
        *   **Flow Method:** Select `[0] Deterministic 8 (D8)`.
        
        *   **Upslope Area:** Save as `watershed_basin.tif`.

---

## 4. Catchment Morphometry Parameters

Morphometry is the quantitative measurement of watershed shapes and networks:

### Drainage Density ($D_d$)

Drainage density measures the total length of streams per unit area:

$$D_d = \frac{\sum L}{A}$$

Where:

*   $\sum L$ = Sum of all stream segment lengths inside the catchment ($\text{km}$).

*   $A$ = Total watershed area ($\text{km}^2$).

*   *Significance:* High drainage density ($D_d > 5\text{ km/km}^2$) indicates impermeable soils, steep slopes, and rapid runoff routing (flashy hydrographs). Low density suggests highly permeable soils and high groundwater infiltration.

### Stream Bifurcation Ratio ($R_b$)

The ratio of the number of streams of a given order ($N_u$) to the number of streams of the next higher order ($N_{u+1}$):

$$R_b = \frac{N_u}{N_{u+1}}$$

*   *Significance:* Typically ranges between $3.0$ and $5.0$ for natural catchments. Higher ratios indicate structurally controlled drainage networks (e.g. geological faults), whereas lower ratios suggest a highly circular catchment with rapid peak flows converging simultaneously at the outlet.

### Relief Ratio ($R_h$)

Measures the overall steepness of the watershed:

$$R_h = \frac{H}{L_b}$$

Where:

*   $H$ = Elevation difference between the highest ridge point and the outlet (relief, $\text{m}$).

*   $L_b$ = Length of the basin parallel to the principal drainage line ($\text{m}$).

---

## 5. Hydrological Significance

Watershed parameters derived via GIS directly configure hydrological routing:

*   **Time of Concentration ($T_c$):** The time required for runoff to travel from the hydraulically most remote point of the watershed to the outlet. GIS catchment slopes and flow lengths are used in the Kirpich equation to estimate $T_c$.

*   **Unit Hydrograph Shape:** Circular catchments route peak flows faster than elongated catchments, producing higher, sharper hydrograph peaks. Sinuosity and form factors calculated in QGIS help estimate peak lag times.


## 6. Data Sources & Acquisition

If you do not have any local elevation data, you can download global Digital Elevation Models (DEMs) for free from the following open-access portals:

*   **Copernicus DEM (30m resolution):** The global standard for topographic routing. Available via the [Copernicus Browser](https://dataspace.copernicus.eu/). Sign up for a free account, search for your Area of Interest (AOI), select the Copernicus DEM (COP-DEM-30) product, and download the tiles in GeoTIFF format.

*   **ALOS World 3D (30m resolution):** High-quality surface model from JAXA. Download from the [JAXA ALOS Portal](https://www.eorc.jaxa.jp/ALOS/en/aw3d30/index.htm).

*   **NASA SRTM DEM (30m resolution):** The legacy shuttle radar topographic mission dataset. Download via [USGS EarthExplorer](https://earthexplorer.usgs.gov/) under the *Digital Elevation* > *SRTM* category, or search on [NASA Earthdata Search](https://search.earthdata.nasa.gov/).
