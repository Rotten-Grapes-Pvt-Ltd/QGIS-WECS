# Hydrological Analysis and Flow Routing

Hydrological analysis models the gravity-driven movement of water across topographic terrain. By routing elevation gradients on raster grids, GIS analysts can calculate water flow pathways, trace stream lines, compute drainage basins, and extract complete river networks. This section covers flow direction algorithms, flow accumulation calculations, stream network thresholding and ordering, and pour point snapping.

---

## 1. Flow Direction Models

Flow direction defines the downhill path that surface water will take from a target center cell to one of its adjacent neighbors. QGIS utilizes three main routing paradigms:

### D8 (Deterministic 8) Single Flow Direction

The D8 algorithm assumes that water from a grid cell drains to exactly one of its eight neighboring cells (cardinal and diagonal) in the direction of the steepest downward elevation gradient.

```text
       D8 FLOW ROUTING DIRECTION ENCODING
       +---------+---------+---------+
       |   32    |   64    |   128   |  <- Power-of-two direction codes
       +---------+---------+---------+
       |   16    |  CELL   |    1    |  <- Water flows to only ONE neighbor
       +---------+---------+---------+
       |    8    |    4    |    2    |  <- Selects the steepest downhill path
       +---------+---------+---------+
```

*   **Direction Codes:**
    
    To store directions as a single integer, cells are coded using powers of two:
    
    *   $1$: East
    
    *   $2$: Southeast
    
    *   $4$: South
    
    *   $8$: Southwest
    
    *   $16$: West
    
    *   $32$: Northwest
    
    *   $64$: North
    
    *   $128$: Northeast

*   **Use and Interpretation:**
    
    Highly efficient for delineating distinct, single-line stream networks in high-relief mountainous catchments. 
    
    *Limitation:* In flat plains or divergent hillsides, D8 creates artificial straight lines and parallel channels because it is constrained to flow directions in multiples of $45^{\circ}$ and cannot split flow.

*   **QGIS Analysis Steps:**
    
    *   Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
    
    *   **Elevation:** Select the conditioned DEM `filled_dem_wang_liu.tif` (generated in the DEM Conditioning chapter).
    
    *   **Method:** Select **\[0\] Deterministic 8 (D8)**.
    
    *   **Flow Directions:** Save as `flow_direction_d8.tif`.
    
    *   Uncheck other outputs and click **Run**.

### Multiple Flow Direction (MFD)

The MFD algorithm distributes downslope runoff across all neighboring cells that are lower in elevation than the target cell. The flow fraction ($f_i$) allocated to neighbor $i$ is proportional to the local slope gradient:

$$f_i = \frac{(\max(0, s_i))^p}{\sum_{j=1}^{8} (\max(0, s_j))^p}$$

Where $s_i$ is the slope to neighbor $i$ and $p$ is a weighting exponent (typically $1.1$ to $8.0$).

*   **Use and Interpretation:**
    
    Highly realistic for modeling soil moisture, land surface wetness (TWI), and overland sheet flow on hillsides. However, it is unsuitable for extracting clean vector river channels because the drainage network disperses across pixels.

*   **QGIS Analysis Steps:**
    
    *   Go to SAGA's **Flow Accumulation (Top-Down)** tool.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Method:** Select **\[4\] Multiple Flow Direction (FD8)**.
    
    *   **Flow Accumulation:** Save as `flow_accumulation_mfd.tif`.
    
    *   Click **Run**.

### D-Infinity ($\text{D}_{\infty}$) Flow Model

The D-Infinity algorithm models flow direction as a continuous angle between $0$ and $2\pi$ radians. The algorithm computes slope vectors across triangular facets formed by the center pixel and its neighbors, routing water along the direction of the steepest facet and allocating flow proportionally between the two adjacent downhill pixels.

*   **Use and Interpretation:**
    
    Combines single-flow convergence in valley channels with continuous dispersion angles on hillsides, avoiding the $45^{\circ}$ discretization bias of the D8 model.

*   **QGIS Analysis Steps:**
    
    *   Go to SAGA's **Flow Accumulation (Top-Down)** tool.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Method:** Select **\[2\] Tarboton's D-Infinity**.
    
    *   **Flow Directions:** Save as `flow_direction_dinf.tif`.
    
    *   Click **Run**.

---

## 2. Flow Accumulation Grids

The Flow Accumulation grid calculates the cumulative number of upstream cells (or total contributing drainage area) draining into each grid cell.

```text
       FLOW DIRECTIONS                 FLOW ACCUMULATION VALUES
       +-----+-----+-----+             +-----+-----+-----+
       |  >  |  v  |  <  |             |  0  |  0  |  0  |  <- Ridge cells (0 inlet)
       +-----+-----+-----+             +-----+-----+-----+
       |     |  v  |     |  ---------> |  0  |  2  |  0  |
       +-----+-----+-----+             +-----+-----+-----+
       |     |  v  |     |             |  0  |  5  |  0  |  <- Stream channel
       +-----+-----+-----+             +-----+-----+-----+
```

The mathematical equation for raster flow accumulation propagation is:

$$A(x,y) = w(x,y) + \sum_{k \in \text{Upstream}} f_k A_k$$

Where $w(x,y)$ is the local cell weight, $A_k$ is the accumulation value of neighbor cell $k$, and $f_k$ is the fraction of flow draining from neighbor cell $k$ to cell $(x,y)$.

### Unweighted Accumulation (Area Delineation)

Each cell is assigned a weight of $1.0$. The resulting accumulation raster values represent the total count of pixels draining through each cell.

*   **Area Conversion:**
    
    $$\text{Basin Contributing Area } (\text{m}^2) = \text{Flow Accumulation Value} \times L_x \times L_y$$
    
    For a standard $30\text{ m}$ grid, multiply accumulation values by $900\text{ m}^2$ (or divide by $1,000,000$ to convert to $\text{km}^2$).

*   **QGIS Analysis Steps:**
    
    *   Navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
    
    *   **Elevation:** `filled_dem_wang_liu.tif`.
    
    *   **Method:** Select **\[0\] Deterministic 8 (D8)**.
    
    *   **Flow Accumulation:** Save as `flow_accumulation_unweighted.tif`.
    
    *   Click **Run**.

### Weighted Accumulation (Runoff Discharge Modeling)

Each cell is multiplied by a secondary raster value (e.g. a spatial rainfall grid or run-off coefficient). The resulting accumulation raster represents actual runoff volumes ($m^3$) rather than cell counts, allowing for spatial discharge modeling.

*   **Use and Interpretation:**
    
    Models water yield and peak flow discharges at basin outlets based on varying soils, slope, and land use weights.

*   **QGIS Analysis Steps:**
    
    *   *Create weight grid:* Use the **Raster Calculator** to multiply a runoff coefficient (e.g. `0.6` for urban, `0.2` for forest) by a rainfall raster. Save as `runoff_weight.tif`.
    
    *   Navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
    
    *   **Elevation:** `filled_dem_wang_liu.tif`.
    
    *   **Weight (optional):** Select `runoff_weight.tif`.
    
    *   **Flow Accumulation:** Save as `flow_accumulation_weighted.tif`.
    
    *   Click **Run**.

---

## 3. Stream Network Delineation and Ordering

To extract river networks from a continuous flow accumulation grid, you must define an accumulation threshold (minimum contributing area) and order the streams by network structure.

### Thresholding Flow Accumulation

Converts a continuous accumulation raster into a binary stream raster representing defined channel lines.

*   **Use and Interpretation:**
    
    *   **Coarse Threshold (High count, e.g. $> 5000$ cells):** Extracts only major river channels with large upstream drainage basins.
    
    *   **Fine Threshold (Low count, e.g. $< 500$ cells):** Extracts minor drainage gullies, headwater streams, and ephemeral runnels.

*   **QGIS Analysis Steps:**
    
    *   Open **Raster** > **Raster Calculator...**.
    
    *   Input the threshold expression (filtering cells with $> 1000$ upstream pixels):
        
        `"flow_accumulation_unweighted@1" >= 1000`
    
    *   Save output as `stream_network_1000.tif`.
    
    *   Click **OK**. Style the output to show only values of $1$ in blue, leaving $0$ transparent.

## 4. Watershed Delineation and Pour Point Snapping

A **Pour Point** (or basin outlet) is the coordinate representing the exit boundary of a drainage network. All water draining through the catchment exits at this point.

### The "One-Pixel Offset" Problem

Stream lines in a flow accumulation grid are only one pixel wide. If a pour point is placed manually and is offset by even a single pixel from the channel path, the local flow accumulation value drops to near-zero. Delineating a watershed from this offset point will result in a catchment size of near-zero ($900\text{ m}^2$) instead of the entire upstream basin.

```text
       POUR POINT OFFSET DISASTER
       +-----+-----+-----+
       | 520 |  3  |  1  |  <- Stream channel path (elevations 520 -> 25 -> 2)
       +-----+-----+-----+
       |  0  |  X  |  0  |  <- X represents manual pour point placement (offset!)
       +-----+-----+-----+
       |  1  |  2  | 980 |  <- Delineation results in only X's cell accumulating!
       +-----+-----+-----+
```

*   **The Snapping Solution:**
    
    The Snap Pour Points tool searches a local window (e.g. $3 \times 3$ or $5 \times 5$ pixels) around the user's point, locates the pixel with the highest flow accumulation, and moves the pour point coordinates to the center of that pixel.

*   **QGIS Snapping Steps:**
    
    *   *Create point:* Go to **Layer** > **Create Layer** > **New GeoPackage Layer...**. Set Geometry to **Point**, name the layer `outlet_point.gpkg`, and click **OK**. Add a point near the downstream end of `stream_network_1000.tif` and save editing.
    
    *   Go to **Processing Toolbox** > **QGIS Raster Analysis** > **Snap Pour Point**.
    
    *   **Input Point Layer:** `outlet_point.gpkg`.
    
    *   **Accumulation Raster:** `flow_accumulation_unweighted.tif`.
    
    *   **Search Radius (in map units):** Set to `50` meters.
    
    *   **Snapped Output:** Save as `snapped_outlet.gpkg` and click **Run**.

### Upslope Catchment Delineation

Extracts the watershed basin boundary upslope of the snapped pour point.

*   **QGIS Analysis Steps:**
    
    *   Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Hydrology** > **Upslope Area**.
    
    *   **Target X Coordinate / Target Y Coordinate:** Pick the coordinate points directly from your `snapped_outlet.gpkg` layer.
    
    *   **Elevation:** `filled_dem_wang_liu.tif`.
    
    *   **Flow Method:** Select **Deterministic 8 (D8)**.
    
    *   **Upslope Area:** Save the output as `watershed_basin.tif` and click **Run**.
    
    *   *Interpretation:* The resulting raster contains values of $100$ inside your delineated catchment basin boundary, and NoData elsewhere. Convert this to a vector polygon layer using **Polygonize (Raster to Vector)** for layout design.
