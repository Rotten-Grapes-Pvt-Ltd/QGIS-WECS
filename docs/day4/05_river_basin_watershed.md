# River Basins and Stream Ordering

River basins are organized hierarchically: large continental river basins are composed of multiple nested sub-basins, which are in turn divided into sub-watersheds and localized catchments. To analyze and manage these drainage networks, stream ordering systems are used to classify channel segments based on their branch complexity and upstream network size. This section covers regional basin delineation workflows and the mathematical rules, uses, and QGIS executions of both the Strahler and Shreve stream ordering models.

---

## 1. Regional Catchment and Drainage Basin Delineation

Unlike single-point watershed delineation (which traces upslope areas from a single pour point coordinate), **regional basin delineation** automatically partitions the entire elevation grid into its component catchments and sub-basins. QGIS utilizes SAGA and GRASS algorithms to run this analysis on conditioned elevation models.

### SAGA Channel Network and Drainage Basins

SAGA's **Channel Network and Drainage Basins** tool is the primary SAGA algorithm to partition a DEM. It extracts both a vector stream segment layer and a raster/polygon layer representing all sub-basins draining to every stream junction.

*   **Use and Interpretation:**
    
    Used to build complete, multi-basin databases for regional water management planning. It identifies how the terrain is divided into discrete hydrologic units, allowing catchment-by-catchment water balance modeling.

*   **QGIS Analysis Steps:**
    
    *   Open QGIS. Ensure the local conditioned DEM `filled_dem_wang_liu.tif` and unweighted accumulation grid `flow_accumulation_unweighted.tif` (from Chapter 4) are loaded.
    
    *   Navigate to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Channels** > **Channel Network and Drainage Basins**.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Flow Accumulation:** Select `flow_accumulation_unweighted.tif`.
    
    *   **Threshold:** Set to `1000` cells (defines the minimum contributing area required to initiate a stream channel).
    
    *   **Channels:** Save output as a vector layer `stream_channels_vector.gpkg`.
    
    *   **Drainage Basins:** Save output as a raster grid `sub_basins_raster.tif`.
    
    *   Click **Run**.
    
    *   *Vectorization:* To convert the raster sub-basins to editable polygon layers for mapping, go to **Raster** > **Conversion** > **Polygonize (Raster to Vector)...**. Select `sub_basins_raster.tif` as input and save as `sub_basins_polygons.gpkg`.

### GRASS r.watershed

The `r.watershed` tool is a robust GRASS GIS algorithm designed to handle massive elevation models. It calculates flow direction, accumulation, stream segments, and sub-basin boundaries simultaneously.

*   **Use and Interpretation:**
    
    Ideal for continental or national-scale watershed mapping (such as modeling the entire Koshi or Gandaki river basins in Nepal). It prevents RAM exhaustion by writing temporary segments directly to disk.

*   **QGIS Analysis Steps:**
    
    *   Navigate to the **Processing Toolbox** > **GRASS** > **Raster (r.*)** > **r.watershed**.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Minimum size of exterior watershed basin:** Set to `1000` (represents cell threshold $T$ to initiate a stream).
    
    *   **Unique label for each watershed basin:** Save the output as `grass_sub_basins.tif`.
    
    *   **Stream segments:** Save the output as `grass_streams.tif`.
    
    *   Click **Run**.

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
```

### Mathematical Rules

Let $u$ and $v$ be two stream segments that merge to form a downstream segment $w$. The Strahler order $O_w$ of the downstream segment is determined by:

$$O_w = \begin{cases} O_u + 1 & \text{if } O_u = O_v \\ \max(O_u, O_v) & \text{if } O_u \neq O_v \end{cases}$$

*   **Order 1:** Headwater streams without any upstream tributaries.
*   **Increment Rule:** The stream order increases by $1$ only when two tributaries of the **same order** intersect (e.g. two Order 2 streams merge to form an Order 3 stream).
*   **No-Change Rule:** When streams of different orders merge, the downstream segment retains the higher order of the two tributaries (e.g. an Order 1 stream merging into an Order 3 stream leaves the downstream channel as Order 3).

### Hydrological Application and Horton's Laws

Strahler order is a proxy for stream scale, slope, and ecological conditions. It forms the basis of **Horton's Law of Stream Numbers**, which states that the number of streams of a given order ($N_u$) decreases geometrically as the stream order increases. This relationship is defined by the **Bifurcation Ratio ($R_b$)**:

$$R_b = \frac{N_u}{N_{u+1}}$$

In natural drainage basins, $R_b$ values typically range between $3.0$ and $5.0$.

*   **High $R_b$ ($> 5.0$):** Indicates a highly dissected basin with steep slopes and rapid surface runoff response, signifying high flash flood hazards.
*   **Low $R_b$ ($< 3.0$):** Indicates flat or elongated basins with slower, delayed runoff hydrograph peaks.
*   **Geomorphic Profile:**
    *   **Orders 1-2 (Headwaters):** Steep gradients, high flow velocities, V-shaped valleys, dominant erosion, and low sediment storage.
    *   **Orders 3-5 (Middle reaches):** Moderate slopes, U-shaped valleys, and a balance of erosion and deposition.
    *   **Orders 6+ (Major rivers):** Very low gradients, broad alluvial floodplains, meandering paths, and dominant deposition.

### QGIS Analysis Steps

1.  **Calculate Strahler Order:**
    
    *   Open the **Processing Toolbox** and navigate to **SAGA** > **Terrain Analysis - Channels** > **Stream Order**.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Streams:** Select the SAGA raster stream grid generated from the channel network (or `stream_channels_vector.gpkg` converted to raster).
    
    *   **Method:** Select **\[0\] Strahler**.
    
    *   **Stream Order:** Save the output as `strahler_order.tif`.
    
    *   Click **Run**.

2.  **Style and Visualize in QGIS:**
    
    *   Load the vector channel layer `stream_channels_vector.gpkg` (from Section 1) in your Layers Panel.
    
    *   Open the layer **Properties** and go to **Symbology**.
    
    *   Change the renderer to **Graduated**.
    
    *   **Value:** Select the attribute field containing the stream order (e.g., `ORDER` or `STRAHLER`).
    
    *   **Method:** Select **Size** (to vary line width dynamically).
    
    *   Click **Classify** and manually configure the line widths:
        *   Order 1: Width $= 0.2\text{ mm}$
        *   Order 2: Width $= 0.5\text{ mm}$
        *   Order 3: Width $= 0.9\text{ mm}$
        *   Order 4: Width $= 1.4\text{ mm}$
        *   Order 5+: Width $= 2.0\text{ mm}$
    
    *   *Add Curved Labels:* Under the **Labels** tab in Properties, select **Single Labels** using the stream order value. Go to the **Placement** sub-tab, set placement to **Curved**, check **On line**, and configure a white text buffer ($1.0\text{ mm}$) to ensure legibility. Click **Apply**.

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

*   **Magnitude 1:** Headwater streams.
*   **Additive Rule:** Every time two streams merge, their magnitudes are added together. For example, when a segment of magnitude $2$ merges with a segment of magnitude $3$, the downstream segment is assigned a magnitude of $5$.

### Hydrological Application

*   **Proxy for Discharge:** Unlike Strahler order, which ignores lower-order tributaries entering major channels, Shreve magnitude counts *every* contributing upstream headwater stream. Consequently, Shreve magnitude values correlate strongly with cumulative drainage basin area and average river discharge.
*   **Ecological and Sediment Modeling:** Used in ecological research (such as the River Continuum Concept) to model cumulative nutrient loads, sediment transport capacity, and species distributions along stream networks.

### QGIS Analysis Steps

1.  **Calculate Shreve Magnitude:**
    
    *   Open SAGA's **Stream Order** tool in the Processing Toolbox.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Streams:** Select your stream grid.
    
    *   **Method:** Select **\[1\] Shreve**.
    
    *   **Stream Order:** Save the output as `shreve_magnitude.tif`.
    
    *   Click **Run**.

2.  **Style and Visualize in QGIS:**
    
    *   Select your vector stream channels layer.
    
    *   In the **Symbology** properties, set the renderer to **Graduated**.
    
    *   **Value:** Select the Shreve magnitude field.
    
    *   **Color Ramp:** Choose a sequential ramp (e.g. *Blues*).
    
    *   **Classes:** Click **Classify** and use **Logarithmic Scale** or **Quantiles** classification (since Shreve values increase rapidly downstream). This will make the main river stem appear in deep blue, showing the cumulative discharge loading visually.
