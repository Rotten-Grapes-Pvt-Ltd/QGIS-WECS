# Rainfall Gridding and Spatial Interpolation

Hydrological modeling and water resources planning require continuous spatial grids of precipitation. Because meteorological weather stations capture data at specific point coordinates, spatial interpolation algorithms are used to estimate values across unmeasured cells. This section covers spatial interpolation methods (Inverse Distance Weighting, Thin Plate Splines, Kriging), orographic precipitation modeling using digital elevation models as covariates, and calculating zonal catchment statistics for water balances.

---

## 1. Spatial Interpolation Methods for Point Gauge Data

Interpolation transforms point data into continuous raster surfaces based on Tobler's First Law of Geography: near things are more related than distant things. QGIS and its integrated provider toolboxes (GDAL, SAGA, GRASS) support several gridding algorithms:

```text
       SPATIAL INTERPOLATION ESTIMATION
       
       Station A [20mm]                 Station B [45mm]
              o                                o
               \\                            //
                \\   (Distance d1)           // (Distance d2)
                 \\                        //
                  v                        v
                +----------------------------+
                |     Interpolated Pixel     | <-- Estimates precipitation
                |      (Target Cell Zp)      |     based on proximity
                +----------------------------+
```

### Inverse Distance Weighting (IDW)

IDW is a deterministic method that estimates cell values by taking a weighted average of surrounding gauge records. The weight assigned to a station decreases as the distance to the target pixel increases:

$$Z_p = \frac{\sum_{i=1}^n \frac{Z_i}{(d_i)^k}}{\sum_{i=1}^n \frac{1}{(d_i)^k}}$$

Where:

*   $Z_p$ is the estimated precipitation value at the target pixel.

*   $Z_i$ is the recorded precipitation at station $i$.

*   $d_i$ is the distance from the target pixel to station $i$.

*   $k$ is the distance friction exponent (typically set to $2$, which is the inverse-square law).
    
    *   **Low Exponent (e.g. $k = 1$):** Faraway stations exert significant influence, smoothing the raster surface.
    
    *   **High Exponent (e.g. $k \ge 3$):** Proximity dominates, creating circular "bullseye" artifacts around isolated rain gauges.

### Thin Plate Splines (TPS)

TPS fits a smooth mathematical surface through the control points while minimizing the bending energy of the surface.

*   **Mathematical Concept:**
    
    Analogous to bending a thin, elastic metal sheet so it passes exactly through the height values of the point gauges.
    
    $$E(z) = \iint \left( \left(\frac{\partial^2 z}{\partial x^2}\right)^2 + 2\left(\frac{\partial^2 z}{\partial x \partial y}\right)^2 + \left(\frac{\partial^2 z}{\partial y^2}\right)^2 \right) dx dy$$

*   **Characteristics:**
    
    Produces a smooth, continuous surface. It is useful for mapping regional temperatures.
    
    *Limitation:* In areas with high gradients and sparse gauges, splines can overshoot, predicting values that are lower or higher than any input station (sometimes yielding impossible negative rainfall estimates).

### Kriging (Geostatistical Interpolation)

Kriging is a geostatistical method that uses the spatial correlation structure of point data to calculate weights. It models spatial variance using a **semivariogram**:

$$\gamma(h) = \frac{1}{2N(h)} \sum_{i=1}^{N(h)} (Z(x_i) - Z(x_i + h))^2$$

Where:

*   $\gamma(h)$ is the semivariance at lag distance $h$.

*   $N(h)$ is the number of point pairs separated by distance $h$.

*   $Z(x_i)$ is the value at point $x_i$.

*   **Semivariogram Parameters:**
    
    *   **Nugget:** The Y-intercept, representing measurement error or micro-scale spatial variance below the minimum sampling distance.
    
    *   **Sill:** The asymptotic plateau where the semivariance levels off, representing the total spatial variance of the dataset.
    
    *   **Range:** The lag distance at which the sill is reached. Points separated by distances greater than the range are no longer spatially correlated.

*   **Characteristics:**
    
    Provides both the interpolated estimate and an estimate of the prediction error (kriging variance).
    
    This is the standard geostatistical method for regional weather analysis.

---

## 2. Modeling Orographic Rainfall (Co-Kriging and DEMs)

In mountainous regions like the Himalayas, precipitation is heavily influenced by topography. As moist air masses hit mountain ranges, they rise, cool, and release moisture on the windward slopes (**orographic precipitation**), leaving the leeward side in a dry **rain shadow**.

```text
       OROGRAPHIC PRECIPITATION INTERACTION
       
           Moist Air
            ----->         /\\  (Ridge Divide)
                          /  \\
                Rain     /    \\ Rain Shadow (Dry)
               ///      /      \\
              ///      /        \\
       ===============+          +===============
       [Windward Face]            [Leeward Face]
```

### The Elevation Problem

Weather stations are often located in valleys where they are accessible.

Standard interpolation methods like IDW or Ordinary Kriging only analyze point coordinates, ignoring topography.

This leads to significant underestimation of precipitation on high-altitude mountain slopes.

### Co-Kriging and Regression Kriging

Co-Kriging incorporates a secondary, high-resolution dataset—a Digital Elevation Model (DEM)—as a covariate alongside the point rainfall records.

*   **Regression Kriging Workflow:**
    
    1.  **Calculate Regression:** Perform a linear or polynomial regression between rain gauge values ($P$) and their elevations ($Z$) extracted from the DEM:
        
        $$P_{\text{regression}} = a \times Z + b$$

    2.  **Extract Residuals:** Compute the difference (residual) between the actual recorded rainfall and the regression-predicted value at each station:
        
        $$\epsilon_{\text{station}} = P_{\text{actual}} - P_{\text{regression}}$$

    3.  **Interpolate Residuals:** Interpolate the residuals (noise) across the grid using Ordinary Kriging to create a residual raster $\epsilon(x,y)$.

    4.  **Assemble Final Grid:** Calculate the final precipitation surface by applying the regression equation to the DEM grid and adding the interpolated residual raster:
        
        $$P(x,y) = (a \times \text{DEM}(x,y) + b) + \epsilon(x,y)$$
        
        This produces a rainfall grid that reflects local windward/leeward and elevation dynamics.

---

## 3. Zonal Catchment Statistics for Water Balances

Once a spatial rainfall grid is created, analysts must calculate the total volume of water entering each sub-catchment.

*   **Zonal Statistics Method:**
    
    An overlay operation that summarizes raster cell values within the boundaries of vector polygon zones (e.g. catchments).
    
    The tool extracts all cell centers that fall within a polygon and calculates descriptive statistics: Mean, Sum, Minimum, Maximum, and Standard Deviation.

*   **The Volumetric Basin Equation:**
    
    To convert the mean precipitation depth ($mm$) calculated by Zonal Statistics into a volumetric input ($m^3$) for water balance models:
    
    $$V_{\text{precipitation}} = \left( \frac{P_{\text{mean}}}{1000} \right) \times A_{\text{catchment}}$$
    
    Where:
    
    *   $V_{\text{precipitation}}$ is the precipitation volume in cubic meters ($m^3$).
    
    *   $P_{\text{mean}}$ is the mean catchment precipitation depth in millimeters ($mm$).
    
    *   $A_{\text{catchment}}$ is the horizontal projected catchment surface area in square meters ($m^2$).

*   **Watershed Water Balance:**
    
    The volumetric precipitation serves as the primary input ($P$) in the watershed balance equation:
    
    $$P = Q + ET + \Delta S$$
    
    Where $Q$ is surface discharge (runoff), $ET$ is evapotranspiration, and $\Delta S$ is the change in soil and groundwater storage.

---

## 4. Step-by-Step Exercise: Rainfall Gridding in QGIS

We will interpolate point gauge data and extract zonal statistics for sub-catchments.

1.  **Load Input Layers:**
    
    Open QGIS. Load `rain_gauges.shp` (containing precipitation records in the field `precip_mm`) and `sub_basins.gpkg`.
    
    Ensure the project CRS is a metric projection (e.g., **UTM Zone 44N**).

2.  **Run QGIS IDW Interpolation:**
    
    Go to **Processing Toolbox** > **QGIS** > **Raster Analysis** > **IDW Interpolation**.
    
    *   **Vector Layer:** `rain_gauges.shp`.
    
    *   **Interpolation Attribute:** Select `precip_mm`.
    
    *   **Distance Coefficient:** Set to `2` (inverse-square).
    
    *   **Extent:** Set to use the extent of `sub_basins.gpkg`.
    
    *   **Output Raster Size (Pixel Size):** Set Pixel Size X and Y to `30` meters.
    
    *   **Interpolated:** Save as `rainfall_idw.tif`.
    
    Click **Run**.

3.  **Run SAGA Regression Kriging:**
    
    Go to **Processing Toolbox** > **SAGA** > **Grid - Gridding** > **Regression Kriging**.
    
    *   **Points:** `rain_gauges.shp`.
    
    *   **Attribute:** `precip_mm`.
    
    *   **Predictor (Grid):** Select your projected elevation model `projected_dem.tif`.
    
    *   **Grid Output:** Save as `rainfall_kriging_elev.tif`.
    
    Click **Run**.

4.  **Extract Catchment Zonal Statistics:**
    
    Go to **Processing Toolbox** > **Raster Analysis** > **Zonal Statistics**.
    
    *   **Input Raster:** `rainfall_kriging_elev.tif`.
    
    *   **Vector Layer Containing Zones:** `sub_basins.gpkg`.
    
    *   **Output Column Prefix:** Type `rain_`.
    
    *   **Statistics to Calculate:** Check `Mean` and `Sum`.
    
    Click **Run**.

5.  **Calculate Volumetric Input:**
    
    Open the attribute table of `sub_basins.gpkg`.
    
    Click **Field Calculator**.
    
    Create a new decimal field named `precip_m3`.
    
    Input the equation converting mean depth in millimeters to volume:
    
    `("rain_mean" / 1000) * $area`
    
    Click **OK**. The field will populate with the volume of precipitation entering each sub-catchment.
