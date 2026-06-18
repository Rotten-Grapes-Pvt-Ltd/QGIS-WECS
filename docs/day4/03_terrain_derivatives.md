# Primary and Secondary Terrain Derivatives

Terrain derivatives are surface rasters calculated directly from Digital Elevation Models (DEMs). They are divided into **Primary Derivatives** (representing geometry and orientation, such as slope, aspect, curvatures, and hillshade) and **Secondary Derivatives** (representing mathematical combinations of primary metrics to model ecological or hydrological processes, such as the Topographic Wetness Index).

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
    
    Uses only the four orthogonal neighbors, ignoring the diagonals.
    
    It is better for smooth, continuous terrains but is highly sensitive to local pixel noise:
    
    $$\frac{\partial z}{\partial x} = \frac{Z_6 - Z_4}{2 \times L_x}$$
    
    $$\frac{\partial z}{\partial y} = \frac{Z_8 - Z_2}{2 \times L_y}$$

*   **Edge Border Handling:**
    
    Along the boundary of a DEM, the $3 \times 3$ window falls outside the dataset.
    
    Software handles this by either clipping the edge cells (returning NoData), copying the edge cell values outward, or wrapping the raster.

---

## 2. Primary Terrain Derivatives

Primary derivatives capture the geometry of the landscape directly from elevation values.

### Slope (Topographic Gradient)

Measures the maximum rate of change of elevation from a cell to its neighbors.

*   **Mathematical Formula:**
    
    $$\text{Slope} = \arctan \left( \sqrt{\left(\frac{\partial z}{\partial x}\right)^2 + \left(\frac{\partial z}{\partial y}\right)^2} \right)$$

*   **Measurement Units:**
    
    *   **Degrees:** Ranges from $0^{\circ}$ (flat plain) to $90^{\circ}$ (vertical cliff).
    
    *   **Percentage:** Calculated as rise over run:
        
        $$\text{Slope}_{\%} = \sqrt{\left(\frac{\partial z}{\partial x}\right)^2 + \left(\frac{\partial z}{\partial y}\right)^2} \times 100$$
        
        A $45^{\circ}$ slope is equivalent to a $100\%$ slope. Vertical cliffs approach infinity.

*   **Hydrological Role:** Direct controller of overland flow velocity, flow kinetic energy, and soil erosion transport capacity (represented as the LS-factor in soil loss equations).

### Aspect (Slope Orientation)

The horizontal direction of the steepest downward slope, measured as a compass angle clockwise from North ($0^{\circ}$ to $360^{\circ}$).

*   **Mathematical Formula:**
    
    $$\text{Aspect} = 270^{\circ} + \arctan2 \left( \frac{\partial z}{\partial y}, -\frac{\partial z}{\partial x} \right)$$
    
    Flat cells with a slope of $0$ are assigned an aspect value of $-1$.

*   **Aspect Micro-Climates in the Himalayas:**
    
    *   **North-facing slopes ($0^{\circ}-45^{\circ}$ and $315^{\circ}-360^{\circ}$):** Receive lower solar radiation, retaining soil moisture and snowpack for longer periods, which supports denser forests.
    
    *   **South-facing slopes ($135^{\circ}-225^{\circ}$):** Exposed to maximum solar radiation, resulting in rapid snowmelt, high soil evaporation rates, and sparse, arid scrub vegetation.

### Hillshade (Shaded Relief)

A visual simulation representing the illumination of the surface under a virtual light source.

*   **Mathematical Formula:**
    
    $$\text{Hillshade} = 255 \times \left( \cos(\text{Zenith}_{\text{sun}}) \cos(\text{Slope}_{\text{local}}) + \sin(\text{Zenith}_{\text{sun}}) \sin(\text{Slope}_{\text{local}}) \cos(\text{Azimuth}_{\text{sun}} - \text{Aspect}_{\text{local}}) \right)$$
    
    Where $\text{Zenith}_{\text{sun}} = 90^{\circ} - \text{Altitude}_{\text{sun}}$.

*   **Configuration Parameters:**
    
    *   **Azimuth:** The compass angle of the virtual sun. The cartographic standard default is $315^{\circ}$ (Northwest) to prevent optical illusion (pseudoscopic illusion where valleys look like ridges and vice versa).
    
    *   **Altitude:** The angle of the sun above the horizon (standard default is $45^{\circ}$).

---

## 3. Secondary Terrain Derivatives

Secondary derivatives combine primary derivatives to model spatial processes like flow acceleration, divergence, or soil moisture accumulation.

### Profile and Planform Curvatures

Curvatures are the second derivatives of elevation, representing how the rate of change of slope and aspect shifts across the landscape.

*   **Profile Curvature (Slope-wise):**
    
    Calculated parallel to the direction of maximum slope.
    
    It controls the acceleration and deceleration of surface runoff.
    
    *   **Convex Profile Curvature:** Accelerates water flow down the slope, increasing soil erosion.
    
    *   **Concave Profile Curvature:** Decelerates water flow, encouraging sediment deposition and water pooling.

*   **Planform Curvature (Contour-wise):**
    
    Calculated perpendicular to the direction of maximum slope.
    
    It controls the convergence and divergence of surface runoff.
    
    *   **Concave Planform Curvature:** Forces overland flow paths to converge, gathering water into channels.
    
    *   **Convex Planform Curvature:** Forces flow paths to diverge, spreading water out across ridges.

```text
       PROFILE AND PLANFORM CURVATURE EFFECTS ON WATER
       
       [Profile: Runoff Speed]          [Planform: Flow Concentration]
       
       Convex (\ ) -> Accelerates      Convex (\/) -> Diverges Flow
       Concave ( \_) -> Decelerates    Concave (V) -> Converges Flow (Channels)
```

---

## 4. Topographic Wetness Index (TWI)

The **Topographic Wetness Index (TWI)** (also known as the Compound Topographic Index) is a steady-state index used to model soil moisture distribution, shallow groundwater tables, and wetland distribution.

*   **Mathematical Formula:**
    
    $$\text{TWI} = \ln \left( \frac{\alpha}{\tan \beta} \right)$$

*   **Parameter Breakdown:**
    
    *   $\alpha$: The Specific Catchment Area (SCA). It is the upslope contributing drainage area per unit contour length ($A / b$).
        
        It represents the volume of water supplied from upstream cells (units in meters).
    
    *   $\beta$: The local slope angle in radians.
        
        It represents the gravitational force driving water export.
        
        To prevent division by zero in flat areas where $\beta = 0$, a very small positive threshold is added: $\tan \beta \approx \tan(\beta + 10^{-4})$.

*   **TWI Hydrological Application:**
    
    *   **High TWI values:** Occur in flat valley bottoms with large upstream contributing catchments (wetlands, floodplains).
    
    *   **Low TWI values:** Occur on steep slopes and ridge tops with small upstream contributing catchments (dry zones).
    
    *   **Landslide Hazard:** A high TWI value on a steep mountain slope indicates high pore-water pressure, which is a major trigger for slope failure.

---

## 5. Step-by-Step Exercise: Computing Derivatives in QGIS

We will generate slope, hillshade, curvatures, and TWI from a raw DEM in QGIS.

1.  **Project Reprojection:**
    
    Load `raw_dem.tif` in QGIS.
    
    Confirm it is reprojected to a metric Projected Coordinate Reference System (e.g. **UTM Zone 44N**). If not, go to **Raster** > **Projections** > **Warp (Reproject)...** to save it in meters.

2.  **Run GDAL Slope:**
    
    Go to **Raster** > **Analysis** > **Slope...**.
    
    *   **Input Layer:** `projected_dem.tif`.
    
    *   **Slope expressed as percent:** Check this if you want percentages, leave unchecked for degrees.
    
    *   **Slope:** Naming output as `slope_degrees.tif`. Click **Run**.

3.  **Run SAGA Curvatures:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Morphometry** > **Slope, Aspect, Curvature**.
    
    *   **Elevation:** Select `projected_dem.tif`.
    
    *   **Profile Curvature:** Save as `profile_curvature.tif`.
    
    *   **Planform Curvature:** Save as `planform_curvature.tif`.
    
    Click **Run**.

4.  **Compute Topographic Wetness Index (TWI):**
    
    First, ensure you have computed the Specific Catchment Area (SCA) using SAGA **Flow Accumulation (Top-Down)**.
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **Topographic Wetness Index**.
    
    *   **Slope:** Select `slope_degrees.tif` (converted to radians or use the SAGA-calculated slope grid).
    
    *   **Area:** Select your flow accumulation/SCA raster.
    
    *   **Topographic Wetness Index:** Save as `twi_output.tif`.
    
    Click **Run**.
    
    Style `twi_output.tif` with a blue-to-yellow color ramp. The deep blue lines highlight the saturated valley channels.
