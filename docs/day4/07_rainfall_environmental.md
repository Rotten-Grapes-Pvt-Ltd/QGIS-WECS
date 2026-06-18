# Rainfall Gridding and Spatial Interpolation

Hydrological modeling and basin water balances require continuous spatial rainfall surfaces. Since weather stations record rainfall at single point coordinates, spatial interpolation algorithms are used to estimate values across unmeasured cells. This section covers spatial interpolation methods (IDW, Splines, Kriging), orographic rainfall adjustments using DEMs, and zonal catchment statistics.

---

## 1. Spatial Interpolation Methods for Point Gauge Data

QGIS integrates interpolation algorithms under the **Raster** > **Analysis** menu or the Processing Toolbox:

```text
    SPATIAL INTERPOLATION CONCEPTS
    Gauge 1 [25mm]                  Gauge 2 [40mm]
          o                            o
           \                          /
            \                        /
             \--> Interpolated Cell <--/
                  (IDW: Distance decay weight)
                  (Kriging: Statistical correlation)
```

### Inverse Distance Weighting (IDW)

Estimates cell values by averaging nearby gauge records, weighting each gauge based on its inverse distance to the target cell:

$$Z_p = \frac{\sum_{i=1}^n \frac{Z_i}{(d_i)^k}}{\sum_{i=1}^n \frac{1}{(d_i)^k}}$$

Where $Z_p$ is the estimated value, $Z_i$ is the value at gauge $i$, $d_i$ is the distance to gauge $i$, and $k$ is the distance friction exponent (typically $2$).

* **Characteristics:** Simple, fast, and stable. However, it is highly sensitive to extreme values, often creating circular "bullseye" artifacts around isolated rain gauges.

### Thin Plate Splines (TPS)

Fits a smooth, elastic mathematical surface across the point heights, passing exactly through every point control station.

* **Characteristics:** Creates smooth, continuous transitions. Best for modeling regional temperatures, but can produce wild overshoots or negative values in areas with sparse station density.

### Kriging (Geostatistical Interpolation)

An advanced interpolation method that models the spatial correlation structure of the data using a **semi-variogram** to calculate statistical weights.

* **Characteristics:** Provides both the interpolated estimate and an estimate of the prediction error (variance). This is the industry standard for modeling regional precipitation.

---

## 2. Modeling Orographic Rainfall (Co-Kriging and DEMs)

In mountainous catchments (such as the Nepalese Himalayas), rainfall is heavily influenced by terrain elevation. As moisture-laden winds hit mountains, they rise, cool, and dump rain on the windward slopes (**orographic precipitation**), leaving the leeward side in a dry **rain shadow**.

```text
    OROGRAPHIC PRECIPITATION EFFECT
         Moist Wind
           ----->      /\ (Ridge Divide)
                      /  \
             Rain    /    \ Rain Shadow (Dry)
            ///     /      \
           ///     /        \
    ==============+          +==============
    [Windward Side]          [Leeward Side]
```

Standard IDW or Kriging models do not account for elevation, resulting in major rainfall estimation errors in mountain valleys. 

### Co-Kriging (Multivariate Kriging)

Incorporates a secondary, highly resolved raster dataset (a **DEM**) as a covariate alongside the point rain gauge values. The algorithm calculates the statistical correlation between station elevation and recorded rainfall. It applies this relationship across the DEM grid to output a rainfall surface that accurately reflects topographic windward and leeward dynamics.

---

## 3. Zonal Statistics for Catchment Water Balances

Once a spatial rainfall grid (e.g., daily rainfall from an interpolated surface or a CHIRPS dataset) is created, you must calculate the total precipitation volume entering each sub-catchment:

1. Open the **Zonal Statistics** tool in the Processing Toolbox.

2. **Raster Layer:** Select your spatial rainfall grid.

3. **Vector Layer:** Select your sub-catchment boundary polygons.

4. Check **Mean** and **Sum**.

5. Click **Run**. The `Sum` attribute represents the cumulative precipitation depth ($mm$) across the catchment cells.

6. **Calculate Volume:** Open the Attribute Table and use the Field Calculator to convert this depth into a volumetric measurement (cubic meters, $m^3$):
   $$\text{Volume } (m^3) = \left( \frac{\text{Mean Rainfall } (mm)}{1000} \right) \times \text{Catchment Area } (m^2)$$
   This volumetric output is the primary input parameter for water balance calculations.
