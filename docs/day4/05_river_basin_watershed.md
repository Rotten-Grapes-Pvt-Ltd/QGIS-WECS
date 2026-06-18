# River Basins and Stream Ordering

River basins are organized hierarchically: large continental river basins are composed of multiple nested sub-basins, which are in turn divided into sub-watersheds and localized catchments. To analyze and manage these drainage networks, stream ordering systems are used to classify channel segments based on their branch complexity and upstream network size. This section covers basin delineation workflows and the mathematical rules and applications of both the Strahler and Shreve stream ordering models.

---

## 1. Delineation Workflows in GRASS and SAGA

In QGIS, watershed boundaries and stream segments are delineated using advanced SAGA and GRASS algorithms. These tools analyze flow direction and accumulation rasters to partition the landscape into distinct catchments.

### GRASS `r.watershed`

The `r.watershed` tool is a robust algorithm designed to process large elevation models. It calculates flow direction, flow accumulation, stream segments, and sub-basin boundaries simultaneously.

*   **Segment File Processing:**
    
    For very large datasets, `r.watershed` writes temporary segments to disk (`seg` parameter) rather than loading the entire raster into RAM, preventing memory crashes.

*   **Minimum Basin Size (Threshold):**
    
    The user inputs a threshold value $T$ representing the minimum number of cells required to initiate a stream channel.
    
    A smaller threshold increases stream density, extracting minor gullies. A larger threshold simplifies the network, showing only major channels.

*   **Routing Methods:**
    
    Supports both single flow direction (D8) and multiple flow direction (MFD) models for calculating accumulation.

### GRASS `r.water.outlet`

Delineates the boundary of a single catchment draining through a specific outlet pixel (such as a stream gauge or dam site).

*   **Algorithm Method:**
    
    The user inputs the D8 flow direction grid and the coordinates of the target outlet (snapped to the stream channel).
    
    Starting at the outlet cell, the algorithm traces upstream cells recursively along the flow direction pathways, marking every connected cell as $1$ (part of the watershed) and all other cells as NoData.

### SAGA `Upslope Area`

SAGA's **Upslope Area** tool is the primary SAGA equivalent to `r.water.outlet`.

*   **Configuration:**
    
    Takes a project elevation grid, flow direction, and target coordinate points.
    
    It supports multiple flow routing algorithms (Deterministic 8, Multiple Flow Direction, D-Infinity, and Mass Flux).
    
    This flexibility allows analysts to evaluate how different routing methods impact the shape and size of the delineated watershed boundary.

---

## 2. Strahler Stream Ordering

Developed by Arthur Strahler in 1952, this is the standard hierarchical system used to classify stream segments based on their branching structure.

```text
       STRAHLER STREAM ORDERING HIERARCHY
       (1) ---\
               (2) ---\
       (1) ---//        \\
                         (3) ---> Outflow (Order 3)
       (2) -------------//
       
       (1) ----(3) ------------> Outflow (No change, remains Order 3)
```

### Mathematical Rules

Let $u$ and $v$ be two stream segments that merge to form a downstream segment $w$. The Strahler order $O_w$ of the downstream segment is determined by:

$$O_w = \begin{cases} O_u + 1 & \text{if } O_u = O_v \\ \max(O_u, O_v) & \text{if } O_u \neq O_v \end{cases}$$

Where:

*   **Order 1:** Headwater streams without any upstream tributaries.

*   **Increment Rule:**
    
    The stream order increases by $1$ only when two tributaries of the **same order** intersect (e.g. two Order 2 streams merge to form an Order 3 stream).

*   **No-Change Rule:**
    
    When streams of different orders merge, the downstream segment retains the higher order of the two tributaries (e.g. an Order 1 stream merging into an Order 3 stream does not change the order; the downstream channel remains Order 3).

### Hydrological Application and Horton's Laws

Strahler order is a proxy for stream scale, slope, and ecological conditions:

*   **Horton's Law of Stream Numbers:**
    
    The number of streams of a given order ($N_u$) decreases geometrically as the stream order increases.
    
    This relationship is defined by the **Bifurcation Ratio ($R_b$)**:
    
    $$R_b = \frac{N_u}{N_{u+1}}$$
    
    In natural, undisturbed drainage basins, $R_b$ values typically range between $3.0$ and $5.0$.
    
    *   **High $R_b$ (e.g. > 5.0):** Indicates a highly dissected basin with steep slopes and rapid surface runoff response (high flash flood hazard).
    
    *   **Low $R_b$ (e.g. < 3.0):** Indicates flat or elongated basins with slower, delayed runoff peaks.

*   **Geomorphic Properties:**
    
    *   **Orders 1-2 (Headwaters):** Steep gradients, high flow velocities, V-shaped valleys, and dominant erosion.
    
    *   **Orders 3-5 (Middle reaches):** Moderate slopes, U-shaped valleys, and a balance of erosion and deposition.
    
    *   **Orders 6+ (Major rivers):** Very low gradients, broad alluvial floodplains, meandering paths, and dominant deposition.

---

## 3. Shreve Stream Ordering (Stream Magnitude)

Developed by Ronald Shreve in 1966, this is an additive stream ordering system that reflects the total number of headwater tributaries contributing flow to any given channel segment.

```text
       SHREVE STREAM MAGNITUDE (ADDITIVE SYSTEM)
       (1) ---\
               (2) ---\
       (1) ---//        \\
                         (5) ---> Outflow (Magnitude 5)
       (3) -------------//
```

### Mathematical Rules

Let $u$ and $v$ be two stream segments merging to form a downstream segment $w$. The Shreve magnitude $M_w$ of the downstream segment is the sum of the magnitudes of the merging tributaries:

$$M_w = M_u + M_v$$

Where:

*   **Magnitude 1:** Headwater streams.

*   **Additive Rule:**
    
    Every time two streams merge, their magnitudes are added together.
    
    For example, when a segment of magnitude $2$ merges with a segment of magnitude $3$, the downstream segment is assigned a magnitude of $5$.

### Hydrological Application

*   **Proxy for Discharge:**
    
    Unlike Strahler order, which ignores lower-order tributaries, Shreve magnitude counts *every* contributing upstream headwater stream.
    
    Consequently, Shreve magnitude values correlate strongly with cumulative drainage area and average river discharge.

*   **Ecological Modeling:**
    
    Used in ecological research (such as the River Continuum Concept) to model cumulative nutrient loads, sediment transport capacity, and species distributions along stream channels.

---

## 4. Step-by-Step Exercise: Stream Ordering in QGIS

We will extract a stream network from a DEM, calculate both Strahler and Shreve stream orders, and style the stream lines based on their order.

1.  **Delineate Stream Segments:**
    
    Load `filled_dem.tif` and `flow_accumulation.tif` in QGIS.
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Channels** > **Channel Network and Drainage Basins**.
    
    *   **Elevation:** `filled_dem.tif`.
    
    *   **Flow Accumulation:** `flow_accumulation.tif`.
    
    *   **Threshold:** Set to `1000` (represents the minimum cell count to start a stream).
    
    *   **Channels:** Save output as `stream_channels_vector.gpkg`.
    
    *   **Drainage Basins:** Save output as `sub_basins.gpkg`.
    
    Click **Run**.

2.  **Calculate Stream Order:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Channels** > **Stream Order**.
    
    *   **Elevation:** `filled_dem.tif`.
    
    *   **Streams:** `stream_channels_vector.gpkg` (or the SAGA grid output `stream_channels.tif`).
    
    *   **Method:** Select `Strahler`.
    
    *   **Stream Order:** Save output as `strahler_order.tif`. Click **Run**.
    
    Re-run the tool with the method set to `Shreve` and save output as `shreve_order.tif`.

3.  **Style Vector Stream Channels:**
    
    Load `stream_channels_vector.gpkg` in the Layers Panel.
    
    Open the layer **Properties** and go to the **Symbology** tab.
    
    Change the renderer from **Single Symbol** to **Graduated**.
    
    *   **Value:** Select the attribute field containing the stream order (e.g., `ORDER` or `STRAHLER`).
    
    *   **Method:** Select `Size` (to vary line width).
    
    *   **Classes:** Click **Classify** and configure the sizes:
        
        *   Order 1: Width $= 0.2\text{ mm}$
        
        *   Order 2: Width $= 0.4\text{ mm}$
        
        *   Order 3: Width $= 0.8\text{ mm}$
        
        *   Order 4: Width $= 1.2\text{ mm}$
        
        *   Order 5+: Width $= 1.8\text{ mm}$

4.  **Add Labels:**
    
    In layer Properties, go to the **Labels** tab.
    
    Select **Single Labels** and set the Value to the stream order field.
    
    Under the **Placement** sub-tab, set placement to **Curved** and check **On line** to bend the label along the channel path.
    
    Add a text buffer (size $1.0\text{ mm}$, color white) to ensure the labels are readable against the background map. Click **Apply**.
