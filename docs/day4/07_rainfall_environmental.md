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

Where $Z_p$ is the estimated precipitation value at the target pixel, $Z_i$ is the recorded precipitation at station $i$, $d_i$ is the distance from the target pixel to station $i$, and $k$ is the distance friction exponent (typically set to $2$, which is the inverse-square law).

*   **Friction Weight $k$:**
    
    *   **Low Exponent (e.g. $k = 1$):** Faraway stations exert significant influence, smoothing the raster surface.
    
    *   **High Exponent (e.g. $k \ge 3$):** Proximity dominates, creating circular "bullseye" artifacts around isolated rain gauges.

*   **Use and Interpretation:**
    
    Fast and simple to compute. Best for small, dense rain gauge networks in flat terrains. It should not be used in high-relief mountainous catchments where topographical influence dominates.

*   **QGIS Analysis Steps:**
    
    *   Load the point vector layer `rain_gauges.shp` (with field `precip_mm`) and catchment polygon layer `sub_basins_polygons.gpkg` in QGIS.
    
    *   Go to **Processing Toolbox** > **QGIS** > **Raster Analysis** > **IDW Interpolation**.
    
    *   **Vector Layer:** Select `rain_gauges.shp`.
    
    *   **Interpolation Attribute:** Select `precip_mm`.
    
    *   **Distance Coefficient:** Set to `2`.
    
    *   **Extent:** Select **Use Extent from** and select `sub_basins_polygons.gpkg`.
    
    *   **Output Raster Size (Pixel Size):** Set X and Y resolutions to `30` (meters).
    
    *   **Interpolated:** Save the output as `rainfall_idw.tif` and click **Run**.

### Thin Plate Splines (TPS)

TPS fits a smooth mathematical surface through the control points while minimizing the bending energy of the surface, analogous to bending a thin, elastic metal sheet so it passes exactly through the height values of the point gauges:

$$E(z) = \iint \left( \left(\frac{\partial^2 z}{\partial x^2}\right)^2 + 2\left(\frac{\partial^2 z}{\partial x \partial y}\right)^2 + \left(\frac{\partial^2 z}{\partial y^2}\right)^2 \right) dx dy$$

*   **Use and Interpretation:**
    
    Produces a smooth, continuous surface without "bullseyes." Ideal for regional temperature mapping.
    
    *Limitation:* In areas with sparse rain gauges and high gradients, splines can overshoot, predicting impossible negative rainfall estimates in dry valleys.

*   **QGIS Analysis Steps:**
    
    *   Go to the **Processing Toolbox** > **GDAL** > **Raster Analysis** > **Grid (Data metrics)...**.
    
    *   **Input Layer:** `rain_gauges.shp`.
    
    *   **Z value from field:** Select `precip_mm`.
    
    *   **Algorithm:** Select **Thin Plate Spline**.
    
    *   **Grid:** Save as `rainfall_tps.tif` and click **Run**.

### Kriging (Geostatistical Interpolation)

Kriging is a geostatistical method that uses the spatial correlation structure of point data to calculate weights. It models spatial variance using a **semivariogram**:

$$\gamma(h) = \frac{1}{2N(h)} \sum_{i=1}^{N(h)} (Z(x_i) - Z(x_i + h))^2$$

Where $\gamma(h)$ is the semivariance at lag distance $h$, $N(h)$ is the number of point pairs separated by distance $h$, and $Z(x_i)$ is the value at point $x_i$.

*   **Semivariogram Parameters:**
    
    *   **Nugget:** The Y-intercept, representing measurement error or micro-scale spatial variance below the minimum sampling distance.
    
    *   **Sill:** The asymptotic plateau where the semivariance levels off, representing the total spatial variance of the dataset.
    
    *   **Range:** The lag distance at which the sill is reached. Points separated by distances greater than the range are no longer spatially correlated.

*   **Use and Interpretation:**
    
    Kriging is the standard geostatistical method for weather mapping. It provides both the interpolated estimate and an estimate of the prediction error (kriging variance), showing where new gauges are needed.

*   **QGIS Analysis Steps:**
    
    *   Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Gridding** > **Kriging (Ordinary)**.
    
    *   **Points:** Select `rain_gauges.shp`.
    
    *   **Attribute:** Select `precip_mm`.
    
    *   **Model Type:** Select **\[1\] Spherical** or **\[0\] Exponential** based on your semivariogram fit.
    
    *   **Grid:** Save output as `rainfall_kriging_ordinary.tif`.
    
    *   Click **Run**.

---

## 2. Modeling Orographic Rainfall (Co-Kriging and Regression Kriging)

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
```

### The Elevation Bias Problem

Weather stations are often located in accessible valleys. Standard interpolation methods like IDW or Ordinary Kriging only analyze point coordinates, ignoring topography. This leads to significant underestimation of precipitation on high-altitude mountain slopes.

### Regression Kriging with elevation covariates

Regression Kriging incorporates a Digital Elevation Model (DEM) as a covariate alongside the point rainfall records:

1.  **Calculate Regression:** Perform a linear regression between rain gauge values ($P$) and their elevations ($Z$) extracted from the DEM:
    
    $$P_{\text{regression}} = a \times Z + b$$

2.  **Extract Residuals:** Compute the difference (residual) between the actual recorded rainfall and the regression-predicted value at each station:
    
    $$\epsilon_{\text{station}} = P_{\text{actual}} - P_{\text{regression}}$$

3.  **Interpolate Residuals:** Interpolate the residuals (noise) across the grid using Ordinary Kriging to create a residual raster $\epsilon(x,y)$.

4.  **Assemble Final Grid:** Calculate the final precipitation surface by applying the regression equation to the DEM grid and adding the interpolated residual raster:
    
    $$P(x,y) = (a \times \text{DEM}(x,y) + b) + \epsilon(x,y)$$

*   **Use and Interpretation:**
    
    Produces a rainfall grid that reflects local windward/leeward and elevation dynamics, which is crucial for water balance calculations in mountain catchments.

*   **QGIS Analysis Steps:**
    
    *   Navigate to **Processing Toolbox** > **SAGA** > **Grid - Gridding** > **Regression Kriging**.
    
    *   **Points:** Select `rain_gauges.shp`.
    
    *   **Attribute:** Select `precip_mm`.
    
    *   **Predictor (Grid):** Select your projected elevation model `filled_dem_wang_liu.tif` (from Chapter 2).
    
    *   **Regression Model:** Select **\[0\] Linear** (or **\[1\] Multiple Linear** if using multiple covariates).
    
    *   **Grid Output:** Save as `rainfall_regression_kriging.tif`.
    
    *   Click **Run**.

---

## 3. Zonal Catchment Statistics for Water Balances

Once a spatial rainfall grid is created, analysts must calculate the total volume of water entering each sub-catchment.

### Zonal Statistics

An overlay operation that summarizes raster cell values within the boundaries of vector polygon zones (e.g. catchments). The tool extracts all cell centers that fall within a polygon and calculates descriptive statistics: Mean, Sum, Minimum, Maximum, and Standard Deviation.

### The Volumetric Basin Equation

To convert the mean precipitation depth ($mm$) calculated by Zonal Statistics into a volumetric input ($m^3$) for water balance models:

$$V_{\text{precipitation}} = \left( \frac{P_{\text{mean}}}{1000} \right) \times A_{\text{catchment}}$$

Where $V_{\text{precipitation}}$ is the precipitation volume in cubic meters ($m^3$), $P_{\text{mean}}$ is the mean catchment precipitation depth in millimeters ($mm$), and $A_{\text{catchment}}$ is the horizontal projected catchment surface area in square meters ($m^2$).

*   **Use and Interpretation:**
    
    Volumetric precipitation serves as the primary input ($P$) in the watershed balance equation:
    
    $$P = Q + ET + \Delta S$$
    
    Where $Q$ is surface discharge (runoff), $ET$ is evapotranspiration, and $\Delta S$ is the change in soil and groundwater storage.

*   **QGIS Analysis Steps:**
    
    *   Go to **Processing Toolbox** > **Raster Analysis** > **Zonal Statistics**.
    
    *   **Input Raster:** Select `rainfall_regression_kriging.tif`.
    
    *   **Vector Layer Containing Zones:** Select the polygon catchment layer `sub_basins_polygons.gpkg`.
    
    *   **Output Column Prefix:** Type `rain_`.
    
    *   **Statistics to Calculate:** Check **Mean** and **Sum**.
    
    *   Click **Run**.
    
    *   *Calculate Volume:* Open the attribute table of `sub_basins_polygons.gpkg`. Open **Field Calculator**, create a new decimal field named `precip_m3`, and enter the conversion equation:
        
        `("rain_mean" / 1000) * $area`
        
    *   Click **OK**. The field will populate with the volume of precipitation entering each sub-catchment.
