# DEM Processing and Conditioning

Raw digital elevation models (DEMs) contain vertical measurements errors, forest canopy artifacts, and coordinate interpolation noise. These errors must be resolved before running any surface water model. This section details how sinks and depressions disrupt flow routing, the mathematics of conditioning algorithms (breaching and filling), and the theory of vector stream burning.

---

## 1. Sinks, Depressions, and the Mathematical Flow Routing Problem

A **Sink** (also known as a pit or closed depression) is a pixel or a continuous group of pixels surrounded entirely by cells of higher elevation values.

```text
       SINK REPRESENTATION IN RASTER GRIDS
       
       [Raw Elevation Grid]             [D8 Flow Direction Routing]
       +-----+-----+-----+              +-----+-----+-----+
       | 120 | 118 | 122 |              |  \  |  v  |  /  |
       +-----+-----+-----+              +-----+-----+-----+
       | 121 | 110 | 119 |  --------->  |  >  |  X  |  <  | <-- Flow terminates here!
       +-----+-----+-----+              +-----+-----+-----+
       | 123 | 115 | 120 |              |  /  |  ^  |  \  |
       +-----+-----+-----+              +-----+-----+-----+
       [110m center cell is a sink]     [X represents undefined routing]
```

### The Mathematics of D8 Flow Routing

In hydrological modeling, surface water routing relies on digital flow path direction engines. The standard method is the **D8 (Eight-Direction) Flow Model**, which routes water from a center cell $Z_0$ to one of its eight neighboring cells $Z_i$ ($i = 1 \text{ to } 8$) along the path of steepest downward slope.

The slope $s$ to each neighbor is calculated as:

$$s = \frac{Z_0 - Z_i}{d}$$

Where:

*   $Z_0$ is the elevation of the central pixel.

*   $Z_i$ is the elevation of the neighboring pixel.

*   $d$ is the horizontal distance between pixel centers.
    
    For orthogonal neighbors (north, south, east, west), $d = L_x$ (the cell resolution, e.g., $30\text{ m}$).
    
    For diagonal neighbors, $d = \sqrt{2} \times L_x$ (e.g., $\approx 42.42\text{ m}$).

### The Routing Obstacle

If a central cell $Z_0$ is lower than all its neighboring cells ($Z_0 \le Z_i$ for all $i$), the slope calculation $s$ yields negative or zero values for all directions.

*   **Undefined Flow Direction:** The routing engine cannot assign a flow direction. The flow path terminates, creating a digital water trap.

*   **Impact on Catchment Delineation:**
    
    *   **Broken Stream Networks:** The calculated river lines terminate abruptly in small pools instead of reaching the basin outlet.
    
    *   **Underestimated Basin Area:** Upstream catchments that drain into the sink are disconnected, distorting the final watershed boundary.
    
    *   **Zero Accumulation:** Zonal statistics and hydrographs for downstream sections drop to zero, failing to represent actual runoff values.

While natural closed depressions exist (e.g., limestone sinkholes in karst terrain, volcanic craters, or tectonic lakes), the vast majority of sinks in $30\text{ m}$ global satellite DEMs are artifacts of data capture.

*   **Radar Speckle & Phase Noise:** Variations in signal reflectance during InSAR capture create local spikes and troughs.

*   **Forest Canopy Bridges:** Dense vegetation lines crossing a narrow canyon block the sensor, creating a "virtual dam" across the channel.

*   **Integer Terracing:** Rounding decimal values to integer values creates artificial flat shelves with zero slope.

---

## 2. DEM Conditioning Algorithms

To establish a continuous flow network, elevation grids must be preprocessed using terrain conditioning algorithms. There are three primary geoprocessing methods:

### Wang and Liu Sink Filling (SAGA)

The SAGA **Fill Sinks (Wang & Liu)** algorithm is the standard tool for resolving depressions. It operates through the following mathematical steps:

1.  **Identify Pit Cells:** Scan the DEM grid to locate all individual sink pixels and multi-pixel depression basins.

2.  **Determine Spill Elevation ($Z_{\text{spill}}$):** Traverse the outer boundaries of each depression basin to locate the lowest elevation cell along the rim where water would naturally spill out.

3.  **Raise Elevation Values:** Set the elevation of all cells inside the depression basin to the spill elevation value:
    
    $$Z_{\text{new}} = \max(Z_{\text{raw}}, Z_{\text{spill}})$$

4.  **Inject Micro-Gradients:**
    
    Simply raising cells to the spill path creates a perfectly flat surface with zero slope.
    
    To ensure a continuous flow path, the algorithm injects a micro-gradient (e.g., $10^{-5}\text{ meters per meter}$) from the interior of the filled basin pointing toward the outlet spill cell, ensuring a valid D8 flow path.

### Planchon and Darboux Sink Filling (GDAL / GRASS)

The Planchon & Darboux algorithm uses a virtual flooding simulation:

1.  **Initialize Grid:** Set the elevation of all non-boundary cells in the DEM to an extremely high value (approaching infinity). Boundary cells (the edges of the raster) are set to their actual raw elevation.

2.  **Flood Simulation:**
    
    The algorithm scans the raster from the outer edges inward.
    
    For each cell, it adjusts the elevation to the maximum of its raw elevation and the elevation of its lowest neighbor plus a small increment.

3.  **Convergence:** The iterations run until no further cell adjustments occur, producing a conditioned surface where every cell drains toward the boundary.

### DEM Breaching (Sliver Cutting)

In flat floodplains or low-slope valley floors, sink filling can create massive, artificial flat zones, distorting drainage channels. Breaching is an alternative method:

*   **Concept:**
    
    Instead of raising the cells inside the sink, breaching lowers the elevation of the blocking cells (the barrier) downstream.
    
    It carves a narrow trench (breach channel) through the barrier to connect the sink to a lower downstream outlet.

*   **Optimization:** The tool search for the path of least resistance (minimizing the volume of excavation) to cut the breach channel.

### Comparison Matrix: Filling vs. Breaching

| Parameter | Sink Filling | DEM Breaching |
| :--- | :--- | :--- |
| **Elevation Change** | Positive (adds height). | Negative (subtracts height). |
| **Slope Preservation** | Flattens valleys; can create wide shelves. | Preserves steep gorge walls and channel slopes. |
| **Volume Impact** | Artificially adds water storage mass. | Artificially removes topographic mass. |
| **Primary Use Case** | Steep headwater catchments with small, isolated pits. | Flat plains, alluvial fans, and urban road embankments. |

---

## 3. Stream Burning and the AGREE Algorithm

Even after resolving sinks, calculated flow lines can deviate from the actual river channels in flat agricultural plains or alluvial valleys due to cell resolution limits. **Stream Burning** forces the flow routing engine to follow known, observed channels by lowering the DEM cells directly beneath a vector river network.

```text
       STREAM BURNING ELEVATION PROFILE (AGREE MODEL)
       
       DEM Surface Elevation ----------\                 //----------
                                       \\  Smooth Buffer//
                                        \\   /-----\   //
                                         \\ /       \\ //
                                          ||  Sharp  ||  <-- TRENCH BURNT
                                          ||  Buffer ||      (Forces flow path)
                                          ++---------++
```

### The AGREE Algorithm Method

Developed at the University of Texas at Austin, the AGREE algorithm processes the DEM using a vector stream network:

1.  **Rasterize Vector Streams:**
    
    Convert the vector river lines into a grid aligning perfectly with the DEM cells.

2.  **Define Buffer Widths:**
    
    *   **Sharp Buffer ($w_s$):** A narrow corridor representing the immediate stream width (typically $1-2$ pixels).
    
    *   **Smooth Buffer ($w_d$):** A wider corridor representing the valley influence zone (typically $5-10$ pixels).

3.  **Compute Drop Elevations:**
    
    Calculate the distance $d$ from each raster cell to the nearest stream pixel.
    
    *   **Inside Sharp Buffer ($d \le w_s$):**
        
        Drop the elevation of the cells by a fixed sharp value ($dz_s$, e.g., $15\text{ meters}$):
        
        $$Z_{\text{new}} = Z_{\text{raw}} - dz_s$$

    *   **Inside Smooth Buffer ($w_s < d \le w_d$):**
        
        Apply a linear interpolation drop ($dz_d$) from the outer edge of the smooth buffer down to the top of the sharp trench:
        
        $$Z_{\text{new}} = Z_{\text{raw}} - dz_d \times \left( \frac{w_d - d}{w_d - w_s} \right)$$

4.  **Resulting Hydrological Routing:**
    
    This creates a V-shaped valley centered on the actual stream.
    
    The D8 flow routing engine is forced to route water into this valley and follow the correct channel path, preventing flow line deviations.

---

## 4. Practical Exercise: DEM Conditioning in QGIS

We will condition a raw DEM by filling sinks and verifying the depth of the filled depressions using QGIS.

1.  **Load Layers:**
    
    Open QGIS. Load `raw_dem.tif` into the Layers Panel.
    
    Ensure the project CRS is projected (e.g., **UTM Zone 44N**).

2.  **Run Wang & Liu Sink Filling:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Fill Sinks (Wang & Liu)**.
    
    *   **DEM:** Select `raw_dem.tif`.
    
    *   **Minimum Slope (Degree):** Set to `0.01` (injects a micro-gradient to prevent flat areas).
    
    *   **Filled DEM:** Naming output as `filled_dem.tif`.
    
    *   **Flow Directions:** Naming output as `flow_directions_filled.tif`.
    
    Click **Run**.

3.  **Map Depression Depths:**
    
    To isolate and analyze the location and depth of the filled sinks, open **Raster** > **Raster Calculator...**.
    
    Enter the subtraction equation:
    
    `"filled_dem@1" - "raw_dem@1"`
    
    Save output as `sink_depth.tif`. Click **OK**.

4.  **Visualize Sinks:**
    
    Style `sink_depth.tif` using a **Singleband pseudocolor** render type.
    
    Select a sequential red color ramp.
    
    Set the min/max values from `0.1` to `10` meters.
    
    Cells that were raised appear in red, showing the distribution of data noise and virtual dams across your study area.
