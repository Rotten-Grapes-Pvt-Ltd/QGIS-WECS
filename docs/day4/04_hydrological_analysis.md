# Hydrological Analysis and Flow Routing

Hydrological analysis models the gravity-driven movement of water across topographic terrain. By routing elevation gradients on raster grids, GIS analysts can calculate water flow pathways, trace stream lines, compute drainage basins, and extract complete river networks. This section covers flow direction algorithms, flow accumulation calculations, thresholding density, and the pour point snapping geoprocessing workflow.

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

*   **Hydrological Suitability:**
    
    Highly efficient for delineating distinct, single-line stream networks in high-relief mountainous catchments.
    
    *Limitation:* In flat plains or divergent hillsides, D8 creates artificial straight lines and parallel channels because it is constrained to flow directions in multiples of $45^{\circ}$, and cannot split flow.

### Multiple Flow Direction (MFD)

The MFD algorithm distributes downslope runoff across all neighboring cells that are lower in elevation than the target cell. The flow fraction ($f_i$) allocated to neighbor $i$ is proportional to the local slope gradient:

$$f_i = \frac{(\max(0, s_i))^p}{\sum_{j=1}^{8} (\max(0, s_j))^p}$$

Where:

*   $s_i$ is the slope to neighbor $i$.

*   $p$ is a weighting exponent (typically $1.1$ to $8.0$).
    
    A higher value of $p$ concentrates flow towards the steepest slope, while a lower value distributes flow more evenly.

*   **Hydrological Suitability:**
    
    Realistic for modeling soil moisture, land surface wetness, and overland sheet flow on hillsides.
    
    However, it is unsuitable for extracting clean vector river channels because the drainage network disperses across pixels.

### D-Infinity ($\text{D}_{\infty}$) Flow Model

Developed by David Tarboton, the D-Infinity algorithm models flow direction as a continuous angle between $0$ and $2\pi$ radians.

*   **Method:**
    
    The algorithm computes slope vectors across triangular facets formed by the center pixel and its neighbors.
    
    Water is routed along the direction of the steepest facet, allocating flow proportionally between the two adjacent downhill pixels.

*   **Hydrological Suitability:**
    
    Combines single-flow convergence in valley channels with continuous dispersion angles on hillsides, avoiding the $45^{\circ}$ discretization bias of the D8 model.

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

Where:

*   $w(x,y)$ is the local cell weight.

*   $A_k$ is the accumulation value of neighbor cell $k$.

*   $f_k$ is the fraction of flow draining from neighbor cell $k$ to cell $(x,y)$.

### Accumulation Configurations

*   **No-Weight Accumulation:**
    
    Each cell is assigned a weight of $1.0$.
    
    The resulting accumulation raster values represent the total count of pixels draining through each cell.
    
    $$\text{Basin Contributing Area } (\text{m}^2) = \text{Flow Accumulation Value} \times L_x \times L_y$$
    
    For a standard $30\text{ m}$ grid, multiply accumulation values by $900\text{ m}^2$ to calculate basin area.

*   **Weighted Accumulation:**
    
    Each cell is multiplied by a secondary raster value (e.g. a spatial rainfall grid or run-off coefficient).
    
    The resulting accumulation raster represents actual runoff volumes ($m^3$) rather than cell counts, allowing for spatial discharge modeling.

---

## 3. Thresholding and Stream Network Extraction

To extract river networks from a continuous flow accumulation grid, you must define an accumulation threshold (minimum contributing area).

```text
       HIGH THRESHOLD (e.g. 5000 cells)       LOW THRESHOLD (e.g. 500 cells)
       +-------------------------------+      +-------------------------------+
       |             |                 |      |           \   |   /           |
       |             |  <- Coarse      |      |            \  |  /  <- Fine     |
       |             |     (Major only)|      |             \ | /     (Gullies) |
       |             |                 |      |               |               |
```

*   **Coarse Threshold (High cell count):**
    
    Extracts only major river channels with large upstream drainage basins (e.g., threshold set to $5000\text{ pixels}$ or $4.5\text{ km}^2$).
    
    Used for regional basin studies.

*   **Fine Threshold (Low cell count):**
    
    Extracts minor drainage gullies, headwater streams, and ephemeral runnels (e.g., threshold set to $500\text{ pixels}$ or $0.45\text{ km}^2$).
    
    Crucial for localized soil erosion modeling and landslide risk mapping.

*   **Mathematical Extraction:**
    
    $$\text{Stream Raster} = \text{If}(\text{Flow Accumulation} \ge \text{Threshold}, 1, \text{NoData})$$

---

## 4. Pour Points and Basin Outlets

A **Pour Point** (or basin outlet) is the coordinate representing the exit boundary of a drainage network. All water draining through the catchment exits at this point.

*   **Placement:**
    
    Positioned at stream confluences, gauging stations, planned reservoir walls, or where a river crosses administrative boundaries.

*   **The "One-Pixel Offset" Disaster:**
    
    Stream lines in a flow accumulation grid are only one pixel wide.
    
    If a user places a pour point manually and offsets it by even a single pixel from the channel path, the local flow accumulation value drops to near-zero.
    
    Delineating a watershed from this offset point will result in a catchment size of near-zero ($900\text{ m}^2$) instead of the entire upstream basin ($1,000\text{ km}^2$).

*   **The Snapping Solution:**
    
    Analysts must run a **Snap Pour Points** geoprocessing tool.
    
    The tool searches a local window (e.g., $3 \times 3$ or $5 \times 5$ pixels) around the user's point, locates the pixel with the highest flow accumulation, and moves the pour point coordinates to the center of that pixel.

---

## 5. Step-by-Step Exercise: Watershed Delineation in QGIS

We will delineate the watershed boundary draining through a selected stream gauge location.

1.  **Prepare Conditioned Elevation Grid:**
    
    Load `filled_dem.tif` (from your sink filling exercise) in QGIS.

2.  **Calculate Flow Direction and Accumulation:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**.
    
    *   **Elevation:** `filled_dem.tif`.
    
    *   **Method:** Select `D8`.
    
    *   **Flow Accumulation:** Save as `flow_accumulation.tif`.
    
    *   **Flow Directions:** Save as `flow_direction_d8.tif`.
    
    Click **Run**.

3.  **Delineate Binary Stream Network:**
    
    Open **Raster** > **Raster Calculator...**.
    
    Input the threshold expression (filtering cells with $> 1000$ upstream pixels):
    
    `"flow_accumulation@1" >= 1000`
    
    Save output as `stream_network_1000.tif`. Style it to show only values of $1$ in blue, leaving $0$ transparent.

4.  **Create Pour Point Layer:**
    
    Go to **Layer** > **Create Layer** > **New GeoPackage Layer...**.
    
    Set Geometry Type to **Point**. Name the layer `outlet_point.gpkg` and click **OK**.
    
    Toggle editing, click the **Add Point Feature** tool, and place a point near the downstream end of a blue stream line. Toggle editing off and save changes.

5.  **Snap Pour Point:**
    
    Go to **Processing Toolbox** > **QGIS Raster Analysis** > **Snap Pour Point**.
    
    *   **Input Point Layer:** `outlet_point.gpkg`.
    
    *   **Accumulation Raster:** `flow_accumulation.tif`.
    
    *   **Search Radius (in map units):** Set to `50` meters.
    
    *   **Snapped Output:** Save as `snapped_outlet.gpkg`. Click **Run**.

6.  **Delineate Upslope Catchment Area:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Upslope Area**.
    
    *   **Target X Coordinate / Target Y Coordinate:** Pick the coordinate points from your `snapped_outlet.gpkg` layer.
    
    *   **Elevation:** `filled_dem.tif`.
    
    *   **Flow Method:** Select `D8`.
    
    *   **Upslope Area:** Save as `watershed_basin.tif`.
    
    Click **Run**.
    
    The resulting raster contains values of $100$ inside your delineated catchment basin boundary, and NoData elsewhere. Convert this to a vector polygon layer using **Polygonize** for report mapping.
