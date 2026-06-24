# Drought Assessment

Drought is a slow-onset natural hazard characterized by the lack of moisture over extended periods. GIS and satellite remote sensing enable continuous monitoring of meteorological, agricultural, and hydrological drought across regional scales by tracking land surface temperature and plant moisture stress.

---

## 1. Core Objectives

*   **Monitor** vegetation health and vigor anomalies over time.

*   **Calculate** plant canopy water stress indices.

*   **Derive** land surface temperature (LST) variations.

*   **Map** combined drought indices for early warning alerts.

---

## 2. Key GIS Inputs

*   **Multispectral Imagery:** Sentinel-2, Landsat 8/9, or MODIS datasets.
    
    *   *Red & Near-Infrared (NIR) bands:* For vegetation health.
    
    *   *Shortwave Infrared (SWIR) bands:* For canopy moisture.

*   **Thermal Infrared (TIR) Bands:** Landsat TIRS or MODIS thermal bands to calculate surface temperature.

*   **Historical Baselines baselines:** Long-term NDVI and LST statistical averages (usually 10 to 30 years of observations).

---

## 3. Core Spectral Indices for Drought

### Normalized Difference Vegetation Index (NDVI)

Measures chlorophyll greenness and plant health:

$$NDVI = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$

*   *Drought Interpretation:* During agricultural drought, NDVI values drop significantly below the historical average for that specific month, representing vegetation degradation.

### Normalized Difference Water Index (NDWI / NDMI)

Measures liquid water content in vegetation canopies:

$$NDWI_{\text{canopy}} = \frac{\text{NIR} - \text{SWIR}}{\text{NIR} + \text{SWIR}}$$

*   *Drought Interpretation:* Because SWIR is highly absorbed by leaf water, a water-stressed canopy reflects more SWIR, causing the NDWI value to decrease.

---

## 4. Advanced Drought Monitoring Indices

To evaluate drought severity relative to long-term averages, raw index values are normalized:

### Vegetation Condition Index (VCI)

VCI normalizes the current NDVI against historical minimum and maximum values for the same time period:

$$VCI = \frac{NDVI - NDVI_{\min}}{NDVI_{\max} - NDVI_{\min}} \times 100$$

Where:

*   $NDVI$ = Current month's NDVI.

*   $NDVI_{\min}, NDVI_{\max}$ = Long-term minimum and maximum NDVI observed for that month over the historical record.

*   *Classification:* VCI values $< 35\%$ indicate severe vegetation drought; values near $100\%$ indicate optimal conditions.

### Temperature Condition Index (TCI)

TCI models moisture stress based on Land Surface Temperature (LST). High temperatures indicate dry soil and stomatal closure in plants:

$$TCI = \frac{LST_{\max} - LST}{LST_{\max} - LST_{\min}} \times 100$$

*   *Logic:* Higher temperatures (closer to $LST_{\max}$) yield lower TCI values, representing higher thermal stress.

### Vegetation Health Index (VHI)

VHI combines VCI and TCI to model overall agricultural drought stress:

$$VHI = \alpha \cdot VCI + (1 - \alpha) \cdot TCI$$

Where:

*   $\alpha$ = Weight parameter (typically set to $0.5$ representing equal influence of greenness and temperature).

---

## 5. Hydrological Significance

*   **Irrigation Management:** Zonal averages of VHI and NDWI calculated across irrigation schemes help water managers allocate reservoir releases to water-stressed agricultural sectors.

*   **Hydrological Forecasts:** Prolonged soil moisture and vegetation droughts lead to low basin runoff coefficients, meaning a future rainfall event will yield less river flow as dry soils absorb initial precipitation.

*   **Early Warning Systems:** Integrated GIS drought maps support disaster response planning, food security forecasting, and drought relief allocation.


## 6. Data Sources & Acquisition

If you do not have satellite imagery to monitor drought stress:

*   **Sentinel-2 Multispectral Imagery (10m - 20m resolution):** Ideal for current, high-resolution agricultural drought monitoring. Download Sentinel-2 Level-2A (Bottom-of-Atmosphere) tiles from the [Copernicus Browser](https://dataspace.copernicus.eu/).

*   **Landsat 8/9 OLI & TIRS (30m - 100m resolution):** Essential for long-term monitoring and land surface temperature (LST) calculations. Download from the [USGS EarthExplorer](https://earthexplorer.usgs.gov/) under the *Landsat* > *Landsat Collection 2 Level-2* category.

*   **MODIS Vegetation Indices (NDVI/EVI, 250m resolution):** Best for regional, long-term time series trend analysis. Download pre-calculated product codes (e.g. MOD13Q1) from the [NASA AppEEARS Portal](https://appeears.earthdatacloud.nasa.gov/).
