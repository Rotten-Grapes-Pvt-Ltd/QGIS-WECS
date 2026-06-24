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

In desktop GIS (QGIS with GRASS GIS or WhiteboxTools), catchment delineation follows a sequential raster processing chain:

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
    
    *   *Tool:* GRASS **r.fill.dir** or WBT **Fill Depressions**.
    
    *   *Logic:* Identifies and raises artificial elevation depressions (sinks) in the DEM to ensure water can flow continuously toward the outer boundary.

2.  **Flow Direction:**
    
    *   *Tool:* GRASS **r.watershed** or WBT **D8 Pointer**.
    
    *   *Logic:* Calculates the direction of steepest descent from each cell to one of its eight neighboring cells. Output is encoded as grid directions (represented as directions/angles or powers of 2).

3.  **Flow Accumulation:**
    
    *   *Tool:* GRASS **r.watershed** or WBT **D8 Flow Accumulation**.
    
    *   *Logic:* Counts the cumulative number of upstream cells draining into each downstream cell. High flow accumulation cells represent natural drainage channels (streams).

4.  **Stream Network Extraction:**
    
    *   *Tool:* Raster Calculator or WBT **Extract Streams**.
    
    *   *Logic:* Thresholds the flow accumulation raster (e.g., all cells where accumulation $> 500\text{ pixels}$) to isolate the stream network.

5.  **Watershed Delineation:**
    
    *   *Tool:* GRASS **r.water.outlet** or WBT **Watershed**.
    
    *   *Logic:* Traces all upstream cells draining to a specified pour point cell (outlet) based on the flow direction grid, generating a boundary polygon.

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
