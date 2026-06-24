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
    
    *   *Input:* Raw DEM raster (`output_hh_utm.tif`).
    
    *   *Output:* Conditioned, depressionless elevation grid (`filled_dem.tif`).
    
    *   *Logic:* Identifies and raises artificial elevation depressions (sinks) in the DEM to ensure water can flow continuously toward the outer boundary.
    
    *   *How to fill the QGIS Form:*
        
        *   **SAGA Fill Sinks (Wang & Liu):**
            
            *   *DEM:* Select `output_hh_utm.tif`.
            
            *   *Minimum Slope:* Set to `0.01` (injects a micro-gradient to force downstream flow).
            
            *   *Filled DEM:* Click the file dialog button and save as `filled_dem.tif`.
        
        *   **WhiteboxTools FillDepressions:**
            
            *   *Input DEM file:* Select `output_hh_utm.tif`.
            
            *   *Flat increment:* Set to `0.001` (to prevent completely flat surfaces).
            
            *   *Output DEM file:* Save as `filled_dem.tif`.

2.  **Flow Direction:**
    
    *   *Input:* Conditioned DEM (`filled_dem.tif`) from **Step 1**.
    
    *   *Output:* Flow Direction grid (`flow_direction.tif`), encoding flow paths.
    
    *   *Logic:* Calculates the direction of steepest descent from each cell to one of its eight neighboring cells. Output is encoded as grid directions (e.g., $1, 2, 4, 8, 16, 32, 64, 128$).
    
    *   *How to fill the QGIS Form:*
        
        *   **SAGA Flow Accumulation (Top-Down):**
            
            *   *Elevation:* Select `filled_dem.tif` (the output from Step 1).
            
            *   *Method:* Select `[0] Deterministic 8 (D8)`.
            
            *   *Flow Directions:* Save as `flow_direction.tif`. (Uncheck other outputs if only direction is needed).
        
        *   **WhiteboxTools D8Pointer:**
            
            *   *Input DEM file:* Select `filled_dem.tif`.
            
            *   *Output pointer file:* Save as `flow_direction.tif`.

3.  **Flow Accumulation:**
    
    *   *Input:* Conditioned DEM (`filled_dem.tif` from **Step 1**) and/or Flow Direction grid (`flow_direction.tif` from **Step 2**).
    
    *   *Output:* Contributing drainage area grid (`flow_accumulation.tif`).
    
    *   *Logic:* Counts the cumulative number of upstream cells draining into each downstream cell. High flow accumulation cells represent natural drainage channels (streams).
    
    *   *How to fill the QGIS Form:*
        
        *   **SAGA Flow Accumulation (Top-Down):**
            
            *   *Elevation:* Select `filled_dem.tif` (from Step 1).
            
            *   *Method:* Select `[0] Deterministic 8 (D8)`.
            
            *   *Flow Accumulation:* Save as `flow_accumulation.tif`.
        
        *   **WhiteboxTools D8FlowAccumulation:**
            
            *   *Input D8 pointer file:* Select `flow_direction.tif` (from Step 2).
            
            *   *Output flow accumulation file:* Save as `flow_accumulation.tif`.

4.  **Stream Network Extraction:**
    
    *   *Input:* Flow Accumulation grid (`flow_accumulation.tif`) from **Step 3**.
    
    *   *Output:* Binary stream channel grid (`stream_network_binary.tif` where stream = 1, land = 0).
    
    *   *Logic:* Thresholds the flow accumulation raster (e.g., all cells where accumulation $> 500\text{ pixels}$) to isolate the stream network.
    
    *   *How to fill the QGIS Form:*
        
        *   **Raster Calculator (QGIS Native):**
            
            *   *Expression:* Type `"flow_accumulation@1" >= 500`.
            
            *   *Output layer:* Save as `stream_network_binary.tif`.
        
        *   **WhiteboxTools ExtractStreams:**
            
            *   *Input D8 flow accumulation file:* Select `flow_accumulation.tif` (from Step 3).
            
            *   *Threshold:* Type `500.0`.
            
            *   *Output stream file:* Save as `stream_network_binary.tif`.

5.  **Watershed Delineation:**
    
    *   *Input:* Flow Direction grid (`flow_direction.tif` from **Step 2**) and target outlet/pour point coordinates.
    
    *   *Output:* Catchment basin boundary grid (`watershed_basin.tif`).
    
    *   *Logic:* Traces all upstream cells draining to a specified pour point cell (outlet) based on the D8 flow direction grid, generating a boundary polygon.
    
    *   *How to fill the QGIS Form:*
        
        *   **SAGA Upslope Area:**
            
            *   *Elevation:* Select `filled_dem.tif` (from Step 1).
            
            *   *Target X Coordinate / Target Y Coordinate:* Use the coordinate selection picker to click on the stream network (`stream_network_binary.tif`) where your outlet sits.
            
            *   *Flow Method:* Select `[0] Deterministic 8 (D8)`.
            
            *   *Upslope Area:* Save as `watershed_basin.tif`.
        
        *   **WhiteboxTools Watershed:**
            
            *   *Input D8 pointer file:* Select `flow_direction.tif` (from Step 2).
            
            *   *Input pour points file:* Select your snapped outlet point vector layer (`snapped_outlet.gpkg`).
            
            *   *Output watershed file:* Save as `watershed_basin.tif`.

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
