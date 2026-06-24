# Reservoir Monitoring

Reservoirs are critical infrastructure for water supply, irrigation, flood control, and hydropower generation. GIS and remote sensing provide cost-effective methods to monitor reservoir surface water area, track storage volume variations, and estimate capacity loss from sedimentation.

---

## 1. Core Objectives

*   **Extract** reservoir surface water area using spectral indices.

*   **Generate** Elevation-Area-Volume (EAV) curves from digital elevation data.

*   **Monitor** storage volume variations over time using satellite observation timeseries.

*   **Estimate** capacity loss and sedimentation rates.

---

## 2. Key GIS Inputs

*   **Multispectral Imagery:** Sentinel-2 or Landsat 8/9 imagery (specifically Green and SWIR bands).

*   **Pre-impoundment DEM or Bathymetric Survey Grid:** Digital representation of the reservoir basin topography before filling.

*   **Water Level Observations (Telemetry):** Tabular data representing daily water levels (m) at the dam wall.

---

## 3. Water Surface Area Extraction (MNDWI)

To map the water extent, we use the **Modified Normalized Difference Water Index (MNDWI)**, which isolates water while suppressing noise from built-up urban zones:

$$MNDWI = \frac{\text{Green} - \text{SWIR}}{\text{Green} + \text{SWIR}}$$

*   *MNDWI Logic:* Water absorbs SWIR light and reflects green light, yielding high positive MNDWI values ($>0.1$). Soil and vegetation yield negative values.

*   *GIS Extraction:* Apply the index in the Raster Calculator, threshold values $> 0.2$, vectorize the result, and calculate polygon area (`$area`).

---

## 4. Developing Elevation-Area-Volume (EAV) Curves

EAV curves are the mathematical foundation of reservoir operations, showing the relationship between water surface level (elevation), pool area, and storage volume:

```text
    RESERVOIR BASIN PROFILE
            ___________________________  Elevation: H_2 (Area A_2)
           /                           \
          /   _______________________   \ Elevation: H_1 (Area A_1)
         /   /                       \   \
        /___/_________________________\___\ Basin Bed
```

1.  **Extract Area at Incremental Elevations:**
    
    *   Using the pre-impoundment DEM raster, run SAGA **Reclassify by Table** at incremental vertical intervals (e.g., every $1\text{m}$).
    
    *   Count the cumulative pixels below each elevation threshold and multiply by cell area to calculate surface area ($A$).

2.  **Calculate Volume ($V$):**
    
    *   Calculate incremental volume between two elevation steps ($H_1$ and $H_2$) using the trapezoidal formula:
        
        $$\Delta V = \frac{H_2 - H_1}{3} \times \left( A_1 + A_2 + \sqrt{A_1 \times A_2} \right)$$
        
    *   Accumulate $\Delta V$ from the lowest bed elevation up to the maximum operating pool level to generate the total volume curve.

3.  **Plot EAV Curves:** Plot elevation on the Y-axis against Surface Area and Storage Volume on the X-axis.

---

## 5. Estimating Volume and Sedimentation Rates

*   **Satellite Storage Estimation:** When in-situ gauge telemetry is unavailable, extract the water surface area from a current satellite scene using MNDWI. Match this area value back to the reservoir's EAV curve to estimate the current storage volume.

*   **Sedimentation Rate Assessment:** Over years, sediment deposits on the reservoir bed, raising the bottom elevation. If a satellite image shows a smaller surface area at a given gauge elevation compared to historical records, the difference represents storage capacity loss. By comparing historical and current EAV curves, engineers can calculate the annual sedimentation volume ($\text{m}^3/\text{year}$).

---

## 6. Hydrological Significance

*   **Water Security Forecasting:** Monitoring weekly storage trends provides early warnings for urban water supply deficits and agricultural droughts.

*   **Hydropower Planning:** Knowing the available water volume and hydraulic head (water level) allows managers to schedule turbine operations and forecast power output.

*   **Watershed Management Feedback:** High reservoir sedimentation rates indicate severe upstream soil erosion, prompting soil conservation works (forestation, contour farming) in the upper catchment.
