# Flood Mapping and Water Resource Applications

Geospatial terrain analysis is crucial for modeling flood hazards, assessing community vulnerability, and planning infrastructure like reservoirs and check dams. This section covers GIS-hydraulic model coupling (HEC-RAS), the Height Above Nearest Drainage (HAND) inundation model, and stage-volume capacity analysis for reservoirs.

---

## 1. Hydraulic Coupling: QGIS and HEC-RAS

Hydraulic modeling simulates the depth, velocity, and extent of water moving through a channel network. We couple GIS with hydraulic software (such as HEC-RAS) in a two-phase workflow to build geometry and map final inundation zones.

### Pre-Processing (Terrain and Geometry Extraction)

To construct a hydraulic model, engineers must extract physical stream geometries from a high-resolution Digital Terrain Model (DTM).

*   **Use and Interpretation:**
    
    GIS coordinates are converted to river geometry coordinates. Digitizing centerlines and bank lines establishes the flow orientation, while cross-sections capture the shape of the channel bed to calculate flow capacity.

*   **QGIS Pre-Processing Steps:**
    
    1.  Load the pre-conditioned DEM `filled_dem_wang_liu.tif` in QGIS.
    
    2.  Create three new vector line layers: `river_centerline.shp`, `river_banks.shp`, and `flow_paths.shp` (representing left and right overbanks). Digitized lines must always point in the downstream flow direction.
    
    3.  Create a vector line layer `cross_sections.shp` representing cross-section cut lines. Draw lines perpendicular to the river flow path, spanning across the channel and active floodplains from left to right looking downstream.
    
    4.  Extract elevation vertices along the cross-sections: Go to the Processing Toolbox > **SAGA** > **Vector <-> Grid** > **Profiles from Line Layer**. Set **Grid** to `filled_dem_wang_liu.tif`, **Line Layer** to `cross_sections.shp`, and click **Run**.
    
    5.  Export these geometries (centerline, banks, and cross-section profiles) as a GIS-formatted XML/SHP file and import them into HEC-RAS to construct the 1D/2D channel geometry model.

### Post-Processing (Inundation Mapping)

Once HEC-RAS runs the flood simulation using design storm discharge rates, the outputs are brought back into QGIS to map water depths and identify flooded structures.

*   **Use and Interpretation:**
    
    Converts simulated water surface heights into an inundation depth raster. Subtracting the ground elevation from the water elevation identifies which homes, roads, and farm fields lie beneath water.

*   **QGIS Post-Processing Steps:**
    
    1.  Export the water surface elevation grid from HEC-RAS as a geotiff (e.g. `water_surface_elevation.tif`).
    
    2.  Import `water_surface_elevation.tif` back into QGIS alongside the bare-earth DTM `filled_dem_wang_liu.tif`.
    
    3.  Go to **Raster** > **Raster Calculator...**.
    
    4.  Enter the depth subtraction formula:
        
        `"water_surface_elevation@1" - "filled_dem_wang_liu@1"`
    
    *   Save output as `flood_depth.tif`. Click **OK**.
    
    *   *Styling:* Open the Symbology panel for `flood_depth.tif`. Select **Singleband pseudocolor**, set the color ramp to **Blues**, and set the min value to `0.01` (to hide unflooded cells). The output displays water depth ranges across the flooded landscape.

---

## 2. The HAND (Height Above Nearest Drainage) Model

**Height Above Nearest Drainage (HAND)** is a terrain model that normalizes topography based on its relative vertical height above the nearest stream channel cell it drains to.

```text
     HAND CONCEPT ELEVATION PROFILE
     
     Terrain cell [A] (Elevation = 100m, HAND = 15m)
            \
             \__ Terrain cell [B] (Elevation = 90m, HAND = 5m)
                \
                 \__ Stream cell [C] (Elevation = 85m, HAND = 0m)
     ===============================================================
     [If water rises by 5m, cell B is flooded; cell A is safe]
```

### The HAND Mathematical Delineation:

$$\text{HAND} = \text{Elevation}_{\text{cell}} - \text{Elevation}_{\text{nearest\_stream}}$$

The flow path routing engine traces down the D8 flow direction network from each land cell to the nearest stream channel cell, subtracting the channel cell elevation from the land cell elevation.

### Use and Interpretation:

Unlike complex hydraulic models, HAND runs fast using only a DEM. A HAND pixel value of $5\text{ m}$ indicates that if the local river level rises by $5\text{ meters}$, that cell will be flooded, making it an excellent tool for rapid flood vulnerability mapping across massive basins.

### QGIS Analysis Steps

1.  **Delineate the Stream Raster:**
    
    Ensure `filled_dem_wang_liu.tif` and the flow accumulation grid `flow_accumulation_unweighted.tif` are loaded.
    
    *   Open **Raster** > **Raster Calculator...**.
    
    *   Input the stream threshold formula:
        
        `ifelse("flow_accumulation_unweighted@1" >= 1000, 1, nodata())`
    
    *   Save output as `stream_network_binary.tif`.

2.  **Calculate HAND in SAGA:**
    
    *   Navigate to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Relative Heights**.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Streams:** Select `stream_network_binary.tif`.
    
    *   **Relative Heights:** Save output as `hand_model.tif`.
    
    *   Click **Run**.
    
    *   *Interpretation:* Style `hand_model.tif` using a classified color ramp. Cells with values of $0-2\text{ m}$ represent high inundation hazard zones (river corridors and low-lying wetlands), while cells $> 5\text{ m}$ represent safe zones.

---

## 3. Reservoir Capacity Stage-Volume Analysis

When planning a hydropower or irrigation dam, engineers must determine the reservoir's capacity (volume) and surface area at varying water heights (stage).

```text
    RESERVOIR STAGE-VOLUME CURVE
    Stage (Height in meters)
      |         /  Surface Area (m2)
      |        /
      |       /   /  Storage Volume (m3)
      |      /   /
      |     /   /
      +----+---+-------
           Volume / Area
```

### Use and Interpretation

Constructs Stage-Area-Volume curves. Engineers use these curves to size the dam's active storage parameters (e.g., determining the height of the spillway needed to store $10\text{ million } m^3$ of water).

### QGIS Analysis Steps

1.  **Delineate the Reservoir Basin Extent:**
    
    Delineate the watershed boundary upstream of the planned dam axis (using SAGA's `Upslope Area` tool from Chapter 4) and save as `reservoir_catchment.tif`. Use the Raster Calculator to clip your DEM:
    
    `"filled_dem_wang_liu@1" * ("reservoir_catchment@1" / 100)`
    
    Save output as `reservoir_dem.tif`.

2.  **Calculate Surface Area and Volume:**
    
    *   Go to **Processing Toolbox** > **QGIS Raster Analysis** > **Raster Surface Volume**.
    
    *   **Input Layer:** Select `reservoir_dem.tif`.
    
    *   **Base Threshold:** Enter the target water surface level (e.g. `480` meters above sea level).
    
    *   **Method:** Select **Count Only Below** (computes the volume of the depression beneath the water stage).
    
    *   **Volume / Area Output:** Save as a table file `volume_stage_480.html`.
    
    *   Click **Run**.
    
    *   *Curve Construction:* Repeat this tool execution at $5\text{-meter}$ height increments (e.g. 450, 455, 460, ..., 500). Extract the **Volume ($m^3$)** and **Area ($m^2$)** outputs from each run, compile them in a spreadsheet, and plot the Stage-Area-Volume curve.
