# Rainfall Spatial Analysis

Rainfall is the primary input driving watershed hydrological cycles. Because precipitation varies across space and time, spatial analysis in GIS is required to convert point observations (rain gauges) or coarse satellite grids into localized precipitation volumes over a catchment.

---

## 1. Core Objectives

*   **Interpolate** point rain gauge measurements to create a continuous rainfall surface.

*   **Extract** catchment-wide average precipitation values from gridded satellite sensors.

*   **Analyze** historical rainfall trends and return periods for engineering designs.

---

## 2. Key GIS Inputs

*   **Point Gauges Layer:** Vector point features containing latitude, longitude, and historical precipitation values (mm).

*   **Gridded Precipitation Products:** Satellite/model rasters such as CHIRPS (0.05° resolution), GPM IMERG (0.1° resolution), or ERA5-Land reanalysis.

*   **Catchment Boundaries:** Watershed vector polygon layer.

---

## 3. Spatial Interpolation Methods

Point observations must be interpolated across space to estimate rainfall in unmonitored zones:

```text
    SPATIAL INTERPOLATION COMPARISON
    +--------------------------------+
    |   o [45mm]      . (Estimate)   |
    |                 [50mm]         |
    |       o [60mm]                 |
    +--------------------------------+
     - IDW: Calculates simple inverse-distance weights.
     - Kriging: Geostatistical semivariogram modeling.
     - Spline: Fits a smooth mathematical surface.
```

### Inverse Distance Weighting (IDW)

Estimates cell values by averaging neighboring gauge values weighted by the inverse of their distance:

$$P_x = \frac{\sum_{i=1}^{n} \frac{P_i}{d_i^p}}{\sum_{i=1}^{n} \frac{1}{d_i^p}}$$

Where:

*   $P_x$ = Estimated precipitation at target cell $x$ (mm).

*   $P_i$ = Precipitation at gauge $i$ (mm).

*   $d_i$ = Distance from gauge $i$ to cell $x$.

*   $p$ = Power parameter (usually $2$, governing distance decay rate).

*   *Limitations:* Sensitive to clustered gauges; prone to "bull's eye" artifacts around isolated stations.

### Kriging

A geostatistical method that uses a semivariogram to capture spatial autocorrelation structures:

$$P_x = \sum_{i=1}^{n} w_i P_i$$

*   *Significance:* Unlike IDW, Kriging calculates the statistical probability of error along with the interpolation surface, making it valuable for scientific confidence evaluation.

*   *Co-Kriging:* Incorporates auxiliary rasters (such as elevation DEMs) to model topographically induced (orographic) rainfall in mountainous zones.

---

## 4. Processing Gridded Satellite Precipitation (Zonal Statistics)

Satellite grids provide continuous global coverage but contain grid cell offsets. GIS is used to query these grids:

1.  **Import Multi-Dimensional Grids:** Load GPM IMERG or CHIRPS NetCDF/HDF5 files using QGIS's **Add Raster Layer** panel. Select the specific precipitation variable band.

2.  **Reproject & Clip:** Run **Warp (Reproject)** to align the coarse satellite coordinate reference system (usually EPSG:4326) with the projected metric coordinate system of the catchment (e.g., UTM Zone 45N).

3.  **Run Zonal Statistics:**
    
    *   *Tool:* Processing Toolbox > **Zonal Statistics**.
    
    *   *Input Vector:* Catchment boundary polygon.
    
    *   *Input Raster:* Satellite precipitation grid.
    
    *   *Calculation:* Choose **Mean** and **Sum**.
    
    *   *Result:* The tool appends the average rainfall depth (mm) and total volume ($\text{m}^3$) directly to the catchment polygon's attribute table.

---

## 5. Hydrological Significance

*   **Areal Rain Depth:** Provides the $P$ value for water balance calculations ($P - ET - Q = \Delta S$).

*   **Design Storm Isohyetal Maps:** Isohyetal contours generated from interpolated rainfall surfaces outline zones of maximum flood vulnerability, which are critical for sizing spillways, drainage canals, and bridges.

*   **Runoff Hydrographs:** Semi-distributed models (like SWAT) run calculations for each sub-basin using the zonal average rainfall extracted via GIS.
