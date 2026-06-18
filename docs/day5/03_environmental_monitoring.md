# Environmental Monitoring Applications

Geospatial indices provide the foundation for tracking long-term environmental processes, drought cycles, land degradation, and urban heat hazards.

---

## 1. Drought Delineation and Agricultural Stress

Drought is a slow-onset hazard that manifests across spectral indices as anomalous decreases in vegetation density and soil moisture.

*   **Vegetation Condition Index (VCI):**
    
    Compares the current NDVI of a pixel to its historical range (minimum and maximum NDVI) recorded for that specific month or week over a long time series (e.g., 20 years).
    
    $$\text{VCI} = \frac{\text{NDVI}_{\text{current}} - \text{NDVI}_{\text{min}}}{\text{NDVI}_{\text{max}} - \text{NDVI}_{\text{min}}} \times 100$$
    
    *VCI Interpretation:*
    
    *   **VCI < 35:** Severe drought conditions.
    
    *   **VCI between 35 and 50:** Moderate drought conditions.
    
    *   **VCI > 50:** Favorable, normal conditions.

*   **Soil Water Index (SWI):**
    
    Uses microwave band scatterometers to model root-zone moisture anomalies.
    
    Allows analysts to identify soil drying trends prior to visible crop damage.

*   **Land Surface Temperature (LST):**
    
    Derived from thermal infrared band data (e.g., Landsat TIRS or Sentinel-3 SLSTR).
    
    Correlates high surface temperatures with water stress, crop failure, and urban heat island effects.

---

## 2. Land Degradation Monitoring (SDG 15.3.1)

United Nations Sustainable Development Goal 15.3.1 defines land degradation based on three primary sub-indicators:

*   **Land Productivity:**
    
    Measures changes in vegetation productivity over time using multi-temporal NDVI/EVI trends.

*   **Land Cover Change:**
    
    Tracks transitions from high-functioning ecological states to degraded states (e.g., forest conversion to barren land).

*   **Soil Organic Carbon (SOC):**
    
    Models changes in carbon storage in the topsoil using field samples and spatial regression.

*   **"One Out, All Out" Principle:**
    
    If any of these three sub-indicators shows degradation, the entire pixel is classified as degraded.

---

## 3. Practical Exercise: Multi-Criteria Drought Hazard Mapping

We will construct a spatial drought hazard index combining NDVI anomaly data, soil texture, and slope gradients.

1.  **Prepare Layers:**
    
    Ensure you have three rasters loaded:
    
    *   `ndvi_anomaly.tif` (reclassified: $1 = \text{wet}$, $2 = \text{normal}$, $3 = \text{dry}$)
    
    *   `soil_drainage.tif` (reclassified: $1 = \text{poorly drained}$, $2 = \text{well drained}$, $3 = \text{excessively drained / sandy}$)
    
    *   `slope_percent.tif` (reclassified: $1 = \text{flat / retains water}$, $2 = \text{moderate}$, $3 = \text{steep / high runoff}$)

2.  **Determine Weights:**
    
    We will assign weights reflecting the influence of each layer on drought susceptibility:
    
    *   NDVI Anomaly Weight: $50\%$ ($0.50$)
    
    *   Soil Drainage Weight: $30\%$ ($0.30$)
    
    *   Slope Weight: $20\%$ ($0.20$)

3.  **Execute Weighted Overlay:**
    
    Open the **Raster Calculator** and input the weighted equation:
    
    `("ndvi_anomaly@1" * 0.50) + ("soil_drainage@1" * 0.30) + ("slope_percent@1" * 0.20)`
    
    Save output as `drought_hazard_index.tif`.

4.  **Categorize Hazard Classes:**
    
    Style the resulting raster into three distinct bins:
    
    *   **Score < 1.8:** Low Drought Susceptibility.
    
    *   **Score 1.8 to 2.4:** Moderate Susceptibility.
    
    *   **Score > 2.4:** High Susceptibility.
