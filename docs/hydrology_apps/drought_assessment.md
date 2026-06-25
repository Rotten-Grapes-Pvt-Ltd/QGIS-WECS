# Drought Assessment

Drought is a slow-onset natural hazard characterized by a deficit of moisture over extended periods. GIS and satellite remote sensing enable continuous monitoring of meteorological, agricultural, and hydrological drought across regional scales by tracking land surface temperature, soil moisture anomalies, and plant canopy water stress.

> [!TIP]
> **Data Sources & Acquisition:**
> If you do not have satellite imagery to monitor drought stress:
> 
> *   **Sentinel-2 Multispectral Imagery (10m - 20m resolution):** Ideal for current, high-resolution agricultural drought monitoring. Download Sentinel-2 Level-2A (Bottom-of-Atmosphere) tiles from the [Copernicus Browser](https://dataspace.copernicus.eu/).
> 
> *   **Landsat 8/9 OLI & TIRS (30m - 100m resolution):** Essential for long-term monitoring and land surface temperature (LST) calculations. Download from the [USGS EarthExplorer](https://earthexplorer.usgs.gov/) under the *Landsat* > *Landsat Collection 2 Level-2* category.
> 
> *   **MODIS Vegetation Indices (NDVI/EVI, 250m resolution):** Best for regional, long-term time series trend analysis. Download pre-calculated product codes (e.g. MOD13Q1) from the [NASA AppEEARS Portal](https://appeears.earthdatacloud.nasa.gov/).

---

## 1. Core Objectives

*   **Compute** vegetation greenness and canopy moisture stress indices.

*   **Extract** historical baseline minimum and maximum vegetation grids.

*   **Normalize** current vegetation states to calculate the Vegetation Condition Index (VCI).

*   **Classify** continuous drought metrics into standard hazard severity zones.

---

## 2. Key GIS Inputs

*   **Current Multi-spectral Imagery:** Sentinel-2 or Landsat 8/9 bands (Red, NIR, and SWIR).

*   **Historical NDVI Grid Stack:** A multi-year collection of NDVI rasters for the target calendar month (typically at least 10–20 years of historical data to establish baselines).

---

## 3. Step-by-Step Drought Assessment Workflow

Agricultural drought monitoring requires computing spectral indicators, comparing them to historical extremes, and reclassifying them into vulnerability alert classes:

![flow_chart](images/drought_assessment/flow_chart.png)

1.  **Calculate Vegetation Greenness Index (NDVI):**
    
    *   **What We Are Doing:** Computing the Normalized Difference Vegetation Index (NDVI) from Near-Infrared (NIR) and Red spectral bands.
    
    *   **Why This Step is Needed:** Healthy vegetation reflects near-infrared light and absorbs visible red light. During a drought, plants lose chlorophyll and reflect less NIR light. NDVI values drop, serving as a primary indicator of vegetation stress.
    
    *   *Input:* Red band and NIR band rasters (e.g. Landsat 8 Band 4 and Band 5, or Sentinel-2 Band 4 and Band 8).
    
    *   *Output:* Continuous NDVI raster (`ndvi.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the NDVI formula:
            `(NIR_band - Red_band) / (NIR_band + Red_band)`
            *(e.g., `("Sentinel2_NIR@1" - "Sentinel2_Red@1") / ("Sentinel2_NIR@1" + "Sentinel2_Red@1")`)*
        
        *   **Output Layer:** Save as `ndvi.tif`.

2.  **Calculate Canopy Moisture Index (NDWI / NDMI):**
    
    *   **What We Are Doing:** Computing the Normalized Difference Water Index (NDWI, sometimes called NDMI) from Near-Infrared (NIR) and Shortwave Infrared (SWIR) bands.
    
    *   **Why This Step is Needed:** Plant leaf water content strongly absorbs SWIR light. Stressed plants with low leaf moisture reflect more SWIR. NDWI maps leaf liquid water content directly, identifying canopy water stress before visible yellowing (NDVI loss) occurs.
    
    *   *Input:* NIR band and SWIR band rasters (e.g. Landsat 8 Band 5 and Band 6, or Sentinel-2 Band 8 and Band 11).
    
    *   *Output:* Canopy water index raster (`ndwi_canopy.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the canopy moisture formula:
            `(NIR_band - SWIR_band) / (NIR_band + SWIR_band)`
            *(e.g., `("Sentinel2_NIR@1" - "Sentinel2_SWIR1@1") / ("Sentinel2_NIR@1" + "Sentinel2_SWIR1@1")`)*
        
        *   **Output Layer:** Save as `ndwi_canopy.tif`.

3.  **Extract Historical NDVI Extremes (SAGA Statistics for Grids):**
    
    *   **What We Are Doing:** Analyzing a multi-year stack of historical NDVI rasters for the current calendar month to calculate the minimum ($NDVI_{\min}$) and maximum ($NDVI_{\max}$) greenness values for each pixel.
    
    *   **Why This Step is Needed:** To evaluate drought severity, we must establish a baseline. Comparing the current NDVI to the absolute minimum and maximum values observed historically for that specific month isolates drought stress from permanent local terrain features (like low soil fertility or rock outcrops).
    
    *   *Input:* Historical stack of monthly NDVI grids (e.g. all month-of-May NDVI grids from 2015 to 2025).
    
    *   *Output:* Minimum NDVI raster (`ndvi_min.tif`) and Maximum NDVI raster (`ndvi_max.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Statistics for Grids**.
        
        *   **Grids:** Click the selection button (`...`) and check all historical monthly NDVI grids in your stack.
        
        *   **Minimum:** Click and save the minimum output grid as `ndvi_min.tif`.
        
        *   **Maximum:** Click and save the maximum output grid as `ndvi_max.tif`.
        
        *   *Note:* Uncheck other statistics (mean, variance, etc.) to optimize execution time.

4.  **Compute Vegetation Condition Index (VCI) (SAGA Grid Calculator):**
    
    *   **What We Are Doing:** Normalizing the current month's NDVI value against the historical range to calculate the VCI percentage raster.
    
    *   **Why This Step is Needed:** The VCI normalizes greenness between 0% (representing the worst historical condition) and 100% (representing the best historical condition). This allows comparison across different vegetation classes and geographical regions.
    
    *   *Input:* Current NDVI (`ndvi.tif`), Minimum NDVI (`ndvi_min.tif`), and Maximum NDVI (`ndvi_max.tif`).
    
    *   *Output:* Normalized VCI grid (`vci.tif` containing values from 0 to 100).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Calculus** > **Grid Calculator**.
        
        *   **Grids:** Select `ndvi.tif` as Grid 1 (`g1`), `ndvi_min.tif` as Grid 2 (`g2`), and `ndvi_max.tif` as Grid 3 (`g3`).
        
        *   **Formula:** Enter: `((g1 - g2) / (g3 - g2)) * 100`
        
        *   **Result:** Save the output grid as `vci.tif`.

5.  **Delineate Drought Severity Zones (SAGA Reclassify Grid Values):**
    
    *   **What We Are Doing:** Reclassifying the continuous 0–100% VCI raster into discrete, standardized drought severity categories.
    
    *   **Why This Step is Needed:** For civil protection, agricultural planning, and water supply adjustments, continuous maps must be converted into clear, actionable warning levels (e.g., Extreme, Severe, Moderate Drought).
    
    *   *Input:* VCI raster (`vci.tif`) from **Step 4**.
    
    *   *Output:* Reclassified integer drought severity map (`drought_severity.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Reclassify Grid Values**.
        
        *   **Grid:** Select `vci.tif`.
        
        *   **Method:** Select `[2] simple range`.
        
        *   **Lookup Table:** Click the lookup table edit button (`...`) and define the ranges:
            
            *   `0` to `20` -> Value: `1` (Extreme Drought)
            
            *   `20` to `35` -> Value: `2` (Severe Drought)
            
            *   `35` to `50` -> Value: `3` (Moderate Drought)
            
            *   `50` to `100` -> Value: `4` (No Drought / Normal)
            
        *   **Reclassified Grid:** Save as `drought_severity.tif`.

---

## 4. Advanced Drought Monitoring Indices

For holistic assessments, greenness indices are paired with thermal data:

### Temperature Condition Index (TCI)

TCI models moisture stress based on Land Surface Temperature (LST). High temperatures indicate dry soil and plant transpiration closure:

$$TCI = \frac{LST_{\max} - LST}{LST_{\max} - LST_{\min}} \times 100$$

*   *TCI Logic:* Higher land temperatures (approaching $LST_{\max}$) yield lower TCI values, indicating high thermal stress.

### Vegetation Health Index (VHI)

VHI combines agricultural canopy greenness (VCI) and thermal land stress (TCI) to estimate overall drought impact:

$$VHI = \alpha \cdot VCI + (1 - \alpha) \cdot TCI$$

*   *Parameters:* The weight factor $\alpha$ is typically set to `0.5`, representing equal influence of plant greenness and soil temperature. VHI $< 35\%$ signals severe crop failure risks.

---

## 5. Hydrological Significance

*   **Irrigation Allocation:** Zonal summaries of VHI and NDWI calculated across irrigation schemes help water managers plan reservoir gate releases to water-stressed agricultural fields.

*   **Runoff Coefficient Reduction:** Prolonged canopy droughts lead to dry soils. A future rain storm falling on dry soils will generate less river runoff as initial volumes are absorbed, reducing reservoir inflows.

*   **Early Warning Systems:** Integrated GIS drought maps support disaster response planning, food security forecasting, and national drought relief allocation.
