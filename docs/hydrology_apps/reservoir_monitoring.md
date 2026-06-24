# Reservoir Monitoring

Reservoirs are critical infrastructure for water supply, irrigation, flood control, and hydropower generation. GIS and remote sensing provide cost-effective methods to monitor reservoir surface water area, track storage volume variations, and estimate capacity loss from sedimentation.

> [!TIP]
> **Data Sources & Acquisition:**
> If you do not have reservoir bathymetry or water level records:
> 
> *   **Reservoir Surface Imagery:** Download Sentinel-2 and Landsat imagery from the [Copernicus Browser](https://dataspace.copernicus.eu/) and [USGS EarthExplorer](https://earthexplorer.usgs.gov/) to map water surface area variations using the MNDWI index.
> 
> *   **Global Reservoirs and Lakes Databases:** Download dam location attributes, volumes, and catchment areas from the [Global Reservoir and Dam (GRanD) database](https://globaldamwatch.org/grand/) or download lake geometry polygons from the [HydroLAKES Database](https://www.hydrosheds.org/products/hydrolakes).
> 
> *   **Satellite Altimetry Water Levels:** Obtain satellite-derived reservoir water level timeseries (derived from radar altimeters) for major global reservoirs from the [Hydroweb Portal](https://hydroweb.theia-land.fr/) or the [USDA G-REALM Portal](https://glam1.gsfc.nasa.gov/).

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

## 3. Step-by-Step Reservoir Monitoring Workflow

Catchment reservoir storage and area monitoring requires extracting surface water from spectral imagery and intersecting it with volumetric bathymetric capacity grids:

![flow_chart](images/reservoir_monitoring/flow_chart.png)

1.  **Compute Water Surface Index (MNDWI or NDWI):**
    
    *   **What We Are Doing:** Calculating a normalized difference water index (like MNDWI) on multi-spectral satellite bands to distinguish water from terrestrial land cover.
    
    *   **Why This Step is Needed:** Standard satellite images show mixed colors. Computing MNDWI uses green and SWIR bands to exploit water's strong reflection of green light and near-complete absorption of shortwave infrared light, yielding high values ($>0.1$) for water and negative values for soil/vegetation.
    
    *   *Input:* Sentinel-2 or Landsat bands (Green and SWIR).
    
    *   *Output:* MNDWI continuous raster grid (`reservoir_mndwi.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the formula:
            `(Green_band - SWIR_band) / (Green_band + SWIR_band)`
            *(e.g., `("Sentinel2_Green@1" - "Sentinel2_SWIR@1") / ("Sentinel2_Green@1" + "Sentinel2_SWIR@1")`)*
        
        *   **Output Layer:** Save as `reservoir_mndwi.tif`.

2.  **Threshold Index into Binary Water Mask:**
    
    *   **What We Are Doing:** Segmenting the continuous MNDWI image into a binary raster where water is 1 and land is 0.
    
    *   **Why This Step is Needed:** MNDWI outputs a range of decimals. To map the actual pool boundary, a threshold (typically between `0.1` and `0.2`) must be applied to isolate pure water pixels from soil and dry vegetation.
    
    *   *Input:* MNDWI raster (`reservoir_mndwi.tif`) from **Step 1**.
    
    *   *Output:* Binary water grid (`water_mask.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the threshold expression:
            `"reservoir_mndwi@1" >= 0.1`
        
        *   **Output Layer:** Save as `water_mask.tif`.

3.  **Convert Raster Mask to Vector Polygon (Vectorization):**
    
    *   **What We Are Doing:** Transforming the raster water pixels into a vector polygon layer and calculating its surface area.
    
    *   **Why This Step is Needed:** To filter out small puddles and compute the exact surface area of the main reservoir pool, the raster water cells must be grouped and polygonized. QGIS can then calculate the geometry area attribute.
    
    *   *Input:* Binary water grid (`water_mask.tif`) from **Step 2**.
    
    *   *Output:* Cleaned vector polygon layer (`reservoir_boundary.gpkg`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Vector** > **Conversion** > **Polygonize (Raster to Vector)...**.
        
        *   **Input Layer:** Select `water_mask.tif`.
        
        *   **Output Field Name:** Keep the default `DN` (Digital Number).
        
        *   **Vectorized:** Save as `water_polygons.gpkg`.
        
        *   *Post-Processing:* Select the main reservoir polygon, delete peripheral sliver polygons, and run the Field Calculator with `$area` to calculate the reservoir surface area in square meters. Export the cleaned result as `reservoir_boundary.gpkg`.

4.  **Calculate Basin Volume from Elevation levels (SAGA Grid Volume):**
    
    *   **What We Are Doing:** Running a volumetric analysis on a pre-impoundment DEM (digital elevation model) to calculate basin capacity below target elevation levels.
    
    *   **Why This Step is Needed:** To build the Elevation-Area-Volume (EAV) capacity table, we must compute the cumulative storage volume and surface area inside the empty reservoir basin at incremental heights (e.g., in 1-meter intervals).
    
    *   *Input:* Projected pre-impoundment DEM (e.g., `basin_dem_utm.tif`).
    
    *   *Output:* Surface area and volume results printed in the log panel.
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Analysis** > **Grid Volume**.
        
        *   **Grid:** Select `basin_dem_utm.tif` (must be in a projected metric CRS like UTM so volume is computed in cubic meters, not degrees).
        
        *   **Method:** Select `[1] Count Only Below Base Level`.
        
        *   **Base Level:** Enter a target elevation value (e.g., `250` for 250m pool height).
        
        *   **Results:** Click Run, then open the **Log** tab in the tool dialog to copy the calculated volume (m³) and surface area (m²). Repeat this for multiple water elevations to build the reservoir capacity table.

5.  **Estimate Reservoir Storage via Curve Matching:**
    
    *   **What We Are Doing:** Intersecting the current satellite-derived water area (from Step 3) with the EAV curve (from Step 4) to determine the active water storage volume.
    
    *   **Why This Step is Needed:** Many remote reservoirs lack operational water level sensors or telemetry. Estimating the surface area from remote sensing and matching it to the EAV curve provides a highly reliable, remote-sensing estimation of active storage.
    
    *   *Input:* Satellite-derived water area ($A_{sat}$ in m²) and the compiled EAV capacity table.
    
    *   *Output:* Current reservoir water depth level (m) and storage volume (m³).
    
    *   *Procedural Steps:*
        
        *   Plot the EAV values in a spreadsheet or database.
        
        *   Interpolate the water level and storage volume associated with the satellite-derived surface area ($A_{sat}$).
        
        *   Track these storage volumes over historical satellite scenes to plot the reservoir's fill-and-drain seasonal variations.

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

*   **Incremental Calculation:**
    
    If the automated SAGA Grid Volume tool is not used, you can calculate the incremental volume between two elevation steps ($H_1$ and $H_2$) using the trapezoidal formula:
    
    $$\Delta V = \frac{H_2 - H_1}{3} \times \left( A_1 + A_2 + \sqrt{A_1 \times A_2} \right)$$
    
    Accumulating $\Delta V$ from the lowest bed elevation up to the maximum operating pool level generates the total volume curve.

---

## 5. Estimating Volume and Sedimentation Rates

*   **Satellite Storage Estimation:** When in-situ gauge telemetry is unavailable, extract the water surface area from a current satellite scene using MNDWI. Match this area value back to the reservoir's EAV curve to estimate the current storage volume.

*   **Sedimentation Rate Assessment:** Over years, sediment deposits on the reservoir bed, raising the bottom elevation. If a satellite image shows a smaller surface area at a given gauge elevation compared to historical records, the difference represents storage capacity loss. By comparing historical and current EAV curves, engineers can calculate the annual sedimentation volume ($\text{m}^3/\text{year}$).

---

## 6. Hydrological Significance

*   **Water Security Forecasting:** Monitoring weekly storage trends provides early warnings for urban water supply deficits and agricultural droughts.

*   **Hydropower Planning:** Knowing the available water volume and hydraulic head (water level) allows managers to schedule turbine operations and forecast power output.

*   **Watershed Management Feedback:** High reservoir sedimentation rates indicate severe upstream soil erosion, prompting soil conservation works (forestation, contour farming) in the upper catchment.
