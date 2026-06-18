# Primary and Secondary Terrain Derivatives

Terrain derivatives are surface rasters calculated directly from Digital Elevation Models (DEMs). They are divided into **Primary Derivatives** (representing geometry and orientation, such as slope, aspect, curvatures, and hillshade) and **Secondary Derivatives** (representing mathematical combinations of primary metrics to model ecological or hydrological processes, such as the Topographic Wetness Index and Stream Power Index).

---

## 1. The Neighborhood Moving Window Concept

Most primary terrain derivatives are calculated using a **$3 \times 3$ Neighborhood Moving Window** algorithm. The algorithm inspects a grid of nine pixels centered on a target pixel ($Z_5$) to compute local rates of change:

```text
       THE 3x3 NEIGHBORHOOD MOVING WINDOW
       +---------+---------+---------+
       |   Z1    |   Z2    |   Z3    |  <- Cell coordinates relative to Z5
       +---------+---------+---------+
       |   Z4    |   Z5    |   Z6    |  <- Z5 is the central target cell (x, y)
       +---------+---------+---------+
       |   Z7    |   Z8    |   Z9    |  <- Cell size Lx and Ly (resolution)
       +---------+---------+---------+
```

To calculate slope and aspect, we must determine the partial derivatives of elevation ($z$) in the horizontal ($x$) and vertical ($y$) directions (denoted as $\frac{\partial z}{\partial x}$ and $\frac{\partial z}{\partial y}$). Two main algorithms are used:

*   **Horn's Algorithm (Default in GDAL/QGIS):**
    
    Weights orthogonal neighbors twice as heavily as diagonal neighbors, making it robust against raster noise:
    
    $$\frac{\partial z}{\partial x} = \frac{(Z_3 + 2Z_6 + Z_9) - (Z_1 + 2Z_4 + Z_7)}{8 \times L_x}$$
    
    $$\frac{\partial z}{\partial y} = \frac{(Z_7 + 2Z_8 + Z_9) - (Z_1 + 2Z_2 + Z_3)}{8 \times L_y}$$
    
    Where $L_x$ and $L_y$ represent the cell width and height in meters.

*   **Zevenbergen & Thorne's Algorithm:**
    
    Uses only the four orthogonal neighbors, ignoring the diagonals. It is better for smooth, continuous terrains but is highly sensitive to local pixel noise:
    
    $$\frac{\partial z}{\partial x} = \frac{Z_6 - Z_4}{2 \times L_x}$$
    
    $$\frac{\partial z}{\partial y} = \frac{Z_8 - Z_2}{2 \times L_y}$$

*   **Edge Border Handling:**
    
    Along the boundary of a DEM, the $3 \times 3$ window falls outside the dataset. Software handles this by either clipping the edge cells (returning NoData), copying the edge cell values outward, or wrapping the raster.

---

## 2. Prerequisites: Workspace and Reprojection Setup

Before performing neighborhood calculations in QGIS, we must ensure the raw elevation raster is in a Projected Coordinate System (meters) to avoid horizontal-vertical unit mismatches (degrees vs. meters).

1.  **Load raw DEM:**
    
    Open QGIS. Load the local elevation raster `output_hh.tif` from the [docs/data/Natural_Earth_quick_start/DEM/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/) folder.

2.  **Reproject to metric coordinates:**
    
    *   Go to **Raster** > **Projections** > **Warp (Reproject)...**.
    
    *   **Input Layer:** `output_hh.tif`.
    
    *   **Source CRS:** `EPSG:4326` (WGS 84).
    
    *   **Target CRS:** Select `EPSG:32645` (WGS 84 / UTM Zone 45N).
    
    *   **Resampling Method:** Select **Bilinear** (smooth interpolation for continuous elevation values).
    
    *   **Reprojected:** Save the file as `output_hh_utm.tif` in your working directory.
    
    *   Click **Run**.
    
    *   *Result:* Remove the original `output_hh.tif` from the canvas. The new layer `output_hh_utm.tif` is now projected in meters, ready for calculations.

---

## 3. Primary Terrain Derivatives

Primary derivatives capture the geometry of the landscape directly from elevation values.

### Slope (Topographic Gradient)

Measures the maximum rate of change of elevation from a cell to its neighbors.

*   **Mathematical Formula:**
    
    $$\text{Slope} = \arctan \left( \sqrt{\left(\frac{\partial z}{\partial x}\right)^2 + \left(\frac{\partial z}{\partial y}\right)^2} \right)$$

*   **Measurement Units:**
    
    *   **Degrees:** Ranges from $0^{\circ}$ (flat plain) to $90^{\circ}$ (vertical cliff).
    
    *   **Percentage:** Calculated as rise over run:
        
        $$\text{Slope}_{\%} = \sqrt{\left(\frac{\partial z}{\partial x}\right)^2 + \left(\frac{\partial z}{\partial y}\right)^2} \times 100$$
        
        A $45^{\circ}$ slope is equivalent to a $100\%$ slope.

*   **Use and Interpretation:**
    
    Controls overland flow velocity and kinetic energy. High slopes generate rapid runoff and are prone to erosion, while low slopes promote ponding and groundwater infiltration.

*   **QGIS Analysis Example:**
    
    *   Go to **Raster** > **Analysis** > **Slope...**.
    
    *   **Input Layer:** Select `output_hh_utm.tif`.
    
    *   **Slope expressed as percent:** Leave unchecked to output in degrees.
    
    *   **Slope:** Save the output as `slope_degrees.tif` and click **Run**.
    
    *   *Interpretation:* Style the output using a singleband pseudocolor ramp (e.g., *YlOrBr*). Dark brown areas represent steep cliffs (high velocity runoff zones), while light yellow areas represent flat basins (ponding zones).

### Aspect (Slope Orientation)

The horizontal direction of the steepest downward slope, measured as a compass angle clockwise from North ($0^{\circ}$ to $360^{\circ}$).

*   **Mathematical Formula:**
    
    $$\text{Aspect} = 270^{\circ} + \arctan2 \left( \frac{\partial z}{\partial y}, -\frac{\partial z}{\partial x} \right)$$
    
    Flat cells with a slope of $0$ are assigned an aspect value of $-1$.

*   **Use and Interpretation:**
    
    Delineates slope micro-climates. North-facing slopes ($0^{\circ}-45^{\circ}$ and $315^{\circ}-360^{\circ}$) receive lower solar radiation, retaining soil moisture and snowpack longer. South-facing slopes ($135^{\circ}-225^{\circ}$) experience rapid snowmelt and high evapotranspiration.

*   **QGIS Analysis Example:**
    
    *   Go to **Raster** > **Analysis** > **Aspect...**.
    
    *   **Input Layer:** Select `output_hh_utm.tif`.
    
    *   **Aspect:** Save the output as `aspect_degrees.tif` and click **Run**.
    
    *   *Interpretation:* Style using a circular color ramp (e.g., *HSV* or *Spectral*). Compasses are classified: North ($0^{\circ}$), East ($90^{\circ}$), South ($180^{\circ}$), and West ($270^{\circ}$). In Himalayan catchments, aspect determines the timing of snowmelt-induced runoff spikes.

### Hillshade & Multi-directional Shading (Shaded Relief)

A visual simulation representing the illumination of the surface under a virtual light source.

*   **Standard Hillshade Formula:**
    
    $$\text{Hillshade} = 255 \times \left( \cos(\text{Zenith}_{\text{sun}}) \cos(\text{Slope}_{\text{local}}) + \sin(\text{Zenith}_{\text{sun}}) \sin(\text{Slope}_{\text{local}}) \cos(\text{Azimuth}_{\text{sun}} - \text{Aspect}_{\text{local}}) \right)$$
    
    Where $\text{Zenith}_{\text{sun}} = 90^{\circ} - \text{Altitude}_{\text{sun}}$.

*   **Multi-directional Hillshade:**
    
    Standard hillshade uses a single light source, leaving half the terrain in dark shadow. Multi-directional hillshading blends light from four azimuth angles ($225^{\circ}$, $270^{\circ}$, $315^{\circ}$, and $360^{\circ}$), revealing subtle features in shadowed zones.

*   **Use and Interpretation:**
    
    Serves as a visual validation layer to detect elevation anomalies (like terracing or steps) and manually map valley lines and structural ridges.

*   **QGIS Analysis Example:**
    
    *   **Standard Hillshade:** Open **Raster** > **Analysis** > **Hillshade...**, set **Input Layer** to `output_hh_utm.tif`, leave azimuth at `315` and altitude at `45`, save as `hillshade_standard.tif`, and click **Run**.
    
    *   **Multi-directional Hillshade:** Go to **Processing Toolbox** > **GDAL** > **Raster Analysis** > **Hillshade**. Select `output_hh_utm.tif`, check **Compute multidirectional shading**, save as `hillshade_multidirectional.tif`, and click **Run**.
    
    *   *Interpretation:* Place `output_hh_utm.tif` on top of `hillshade_multidirectional.tif`. Style the elevation raster with a *Terrain* color ramp, and set its **Blending mode** in the Layer Styling panel to **Multiply** at $45\%$ transparency to produce a stunning 3D physical map.

---

## 4. Secondary Terrain Derivatives

Secondary derivatives combine primary derivatives to model spatial processes like flow acceleration, divergence, or soil moisture accumulation.

### Profile and Planform Curvatures

Curvatures represent how the rate of change of slope and aspect shifts across the landscape.

*   **Profile Curvature (Slope-wise):**
    
    Calculated parallel to the direction of maximum slope. Controls flow acceleration.
    
    *   **Convex Profile Curvature (positive values):** Accelerates water flow down the slope, increasing soil erosion.
    
    *   **Concave Profile Curvature (negative values):** Decelerates water flow, encouraging deposition.

*   **Planform Curvature (Contour-wise):**
    
    Calculated perpendicular to the direction of maximum slope. Controls flow convergence.
    
    *   **Concave Planform Curvature (negative values):** Forces overland flow paths to converge, gathering water into channels.
    
    *   **Convex Planform Curvature (positive values):** Forces flow paths to diverge, spreading water out.

```text
       PROFILE AND PLANFORM CURVATURE EFFECTS ON WATER
       
       [Profile: Runoff Speed]          [Planform: Flow Concentration]
       
       Convex (\ ) -> Accelerates      Convex (\/) -> Diverges Flow
       Concave ( \_) -> Decelerates    Concave (V) -> Converges Flow (Channels)
```

*   **Use and Interpretation:**
    
    Critical for landslide hazard mapping. Zones of concave planform curvature (converging flow) and convex profile curvature (accelerating flow) represent areas of high saturation and runoff velocity, making them highly prone to shallow landsliding.

*   **QGIS Analysis Example:**
    
    *   Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Morphometry** > **Slope, Aspect, Curvature**.
    
    *   **Elevation:** Select `output_hh_utm.tif`.
    
    *   **Method:** Select **\[6\] 9 Pin Bilinear Interpolation (Zevenbergen & Thorne)**.
    
    *   **Profile Curvature:** Save as `profile_curvature.tif`.
    
    *   **Planform Curvature:** Save as `planform_curvature.tif`.
    
    *   Uncheck other optional outputs and click **Run**.
    
    *   *Interpretation:* Open `planform_curvature.tif` and apply a diverging *RdBu* (Red to Blue) color ramp. Blue cells (negative planform values) delineate valley lines where surface runoff naturally concentrates.

### Topographic Wetness Index (TWI)

The **Topographic Wetness Index (TWI)** is a steady-state index used to model soil moisture distribution, shallow groundwater tables, and wetland boundaries.

*   **Mathematical Formula:**
    
    $$\text{TWI} = \ln \left( \frac{\alpha}{\tan \beta} \right)$$
    
    Where $\alpha$ is the Specific Catchment Area (SCA) (upslope drainage area per unit contour length, $A/b$) and $\beta$ is the local slope in radians.

*   **Use and Interpretation:**
    
    High TWI values identify saturated soils, wetlands, and floodplains. Runoff in these zones is generated via saturation-excess (where the water table reaches the surface). Low TWI values identify dry, shedding slopes.

*   **QGIS Analysis Example:**
    
    *   **Step 1 (SCA):** Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Flow Accumulation (Top-Down)**. Set **Elevation** to `output_hh_utm.tif`, set **Method** to **\[4\] Multiple Flow Direction (FD8)**, save **Flow Accumulation** as `flow_accumulation_sca.tif`, and click **Run**.
    
    *   **Step 2 (TWI):** Go to **SAGA** > **Terrain Analysis - Hydrology** > **Topographic Wetness Index**. Set **Slope** to `slope_degrees.tif` (or leave empty to let the tool compute it internally), set **Area** to `flow_accumulation_sca.tif`, save output as `twi_output.tif`, and click **Run**.
    
    *   *Interpretation:* Style `twi_output.tif` using a blue-to-yellow color ramp. High values (deep blue, e.g., TWI $> 12$) represent saturated stream valleys and alluvial floodplains.

### Stream Power Index (SPI)

SPI measures the potential erosive power of overland flowing water at any point on the landscape.

*   **Mathematical Formula:**
    
    $$\text{SPI} = \alpha \times \tan \beta$$

*   **Use and Interpretation:**
    
    Locates where high flow accumulation (large SCA $\alpha$) and steep slopes ($\beta$) occur together. Used to map gully erosion risks and locate where check dams are needed.

*   **QGIS Analysis Example:**
    
    *   Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Stream Power Index**.
    
    *   **Slope:** Select `slope_degrees.tif`.
    
    *   **Area:** Select `flow_accumulation_sca.tif`.
    
    *   **Stream Power Index:** Save as `spi_output.tif` and click **Run**.
    
    *   *Interpretation:* Apply a sequential *YlOrRd* (Yellow to Red) color ramp. Peak red values pinpoint channel incision risks and gully erosion hotspots.

### Terrain Ruggedness Index (TRI)

TRI quantifies the local variation in elevation within a cell's neighborhood.

*   **Mathematical Formula (Riley et al. 1999):**
    
    $$\text{TRI} = \sqrt{\sum_{i=1}^{8} (Z_5 - Z_i)^2}$$
    
    Where $Z_5$ is the elevation of the central pixel and $Z_i$ represents the elevation of the 8 surrounding neighbors.

*   **Riley Terrain Ruggedness Classification Table:**
    
    | TRI Range (meters) | Ruggedness Category |
    | :--- | :--- |
    | $0 - 80\text{ m}$ | Level / Flat |
    | $80 - 116\text{ m}$ | Nearly Level |
    | $116 - 161\text{ m}$ | Slightly Rugged |
    | $161 - 239\text{ m}$ | Moderately Rugged |
    | $239 - 497\text{ m}$ | Highly Rugged |
    | $497 - 958\text{ m}$ | Extremely Rugged |
    | $> 958\text{ m}$ | Severely Rugged |

*   **Use and Interpretation:**
    
    Quantifies hydraulic resistance. High ruggedness values indicate rough surfaces that slow down runoff velocity but create high local turbulence.

*   **QGIS Analysis Example:**
    
    *   Go to **Raster** > **Analysis** > **Ruggedness Index (TRI)...**.
    
    *   **Input Layer:** Select `output_hh_utm.tif`.
    
    *   **Ruggedness:** Save output as `tri_ruggedness.tif` and click **Run**.
    
    *   *Interpretation:* Use the Raster Calculator to isolate highly rugged valleys (`"tri_ruggedness@1" > 239`). These represent geomorphologically active areas with high resistance to overland flow.

### Topographic Position Index (TPI)

TPI measures the elevation difference between a central pixel and the average elevation of its surrounding neighborhood within a user-defined search radius $R$.

*   **Mathematical Formula:**
    
    $$\text{TPI} = Z_5 - \mu_R$$
    
    Where $Z_5$ is the elevation of the central pixel and $\mu_R$ is the mean elevation of all pixels within a window of radius $R$.

*   **Landform Classification:**
    
    *   **Positive TPI ($\text{TPI} > \text{Threshold}$):** Ridges, hills, or peaks (the cell is higher than its neighbors).
    
    *   **Negative TPI ($\text{TPI} < -\text{Threshold}$):** Valleys, canyons, or depressions (the cell is lower than its neighbors).
    
    *   **Near-Zero TPI ($-\text{Threshold} \le \text{TPI} \le \text{Threshold}$):** Flat plains or uniform slopes.

*   **Use and Interpretation:**
    
    Automates landform classification. Hydrologists use negative TPI to extract valley networks and positive TPI to delineate ridge boundaries.

*   **QGIS Analysis Example:**
    
    *   Go to **Raster** > **Analysis** > **Topographic Position Index (TPI)...**.
    
    *   **Input Layer:** Select `output_hh_utm.tif`.
    
    *   **TPI:** Save output as `tpi_landforms.tif` and click **Run**.
    
    *   *Interpretation:* Apply a classified color scheme to the output. Large positive values delineate mountain peaks and ridges, while negative values outline canyon beds and stream incisions.
