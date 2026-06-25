# Water Quality Assessment

Satellite remote sensing enables the continuous monitoring of water quality parameters across lakes, reservoirs, and rivers. By analyzing the optical reflectance signatures of water bodies, GIS specialists can map turbidity, Total Suspended Solids (TSS), and chlorophyll-a concentrations to identify pollution sources, monitor agricultural runoff, and track harmful algal blooms.

> [!TIP]
> **Data Sources & Acquisition:**
> To perform satellite-based water quality monitoring:
> 
> *   **Sentinel-2 Level-2A (BOA) Reflectance:** Download pre-processed Bottom-of-Atmosphere (BOA) surface reflectance tiles in GeoTIFF format from the [Copernicus Browser](https://dataspace.copernicus.eu/). BOA imagery is mandatory because it removes atmospheric scattering noise which can skew water reflectance measurements.
> 
> *   **Landsat 8/9 Level-2 Surface Reflectance:** Download surface reflectance grids from [USGS EarthExplorer](https://earthexplorer.usgs.gov/).
> 
> *   **UNEP GEMStat (Global Water Quality Database):** Sourced from the United Nations Environment Programme, this database offers global in-situ freshwater quality data. Request historical dataset access at the [GEMStat Portal](https://gemstat.org/) to obtain matching ground-truth measurements.

---

## 1. Core Objectives

*   **Isolate** water bodies using spectral NDWI masking to eliminate terrestrial interference.

*   **Calculate** index indicators for Chlorophyll-a and Total Suspended Solids (TSS).

*   **Extract** satellite index values at in-situ ground sampling locations.

*   **Perform** empirical regression calibration to convert raw index ratios into calibrated concentration maps.

---

## 2. Key GIS and Tabular Inputs

*   **Multispectral Bands:** Aligned Sentinel-2 bands: Green (B3), Red (B4), Red-Edge (B5), and Near-Infrared (B8).

*   **Sampling Coordinates:** Vector point layer (`sampling_stations.gpkg`) containing GPS coordinates and corresponding laboratory-tested water quality measurements.

---

## 3. Step-by-Step Water Quality Assessment Workflow

Mapping water quality requires masking, index calculations, spatial attribute joins, regression modeling, and grid calibration:

```text
  [ Sentinel-2 Reflectance Bands ]               [ GPS In-Situ Station Points ]
    │                     │                                   │
    ▼ (NDWI Mask)         ▼ (Calculators)                     │
  [ Water Mask ] ──> [ NDCI & TSS Indices ]                   │
                           │                                  │
                           ▼                                  ▼
             (SAGA Add Grid Values to Points) ◄───────────────┘
                           │
                           ▼
            [ Combined Attribute Table ]
                           │
                           ▼ (Regression Scatterplot in Excel/Python)
               [ Calibration Equations ]
                           │
                           ▼ (QGIS Raster Calculator)
            [ Calibrated Concentration Maps ]
```

### Step 1: Delineate Water Surface Mask (NDWI Thresholding)

*   **What We Are Doing:** Creating a binary raster mask that isolates the open water surface, setting all land, forest, and urban pixels to NoData.
    
*   **Why This Step is Needed:** Terrestrial features have high reflectance in visible and infrared bands. Masking out land pixels prevents land features from corrupting the index calculation range and skewing the final water map statistics.
    
*   *Input:* Sentinel-2 Green (B3) and NIR (B8) bands.
    
*   *Output:* Binary water mask raster (`water_mask.tif`).
    
*   *How to Fill the Form in QGIS:*
    
    *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
    
    *   **Expression:** Enter the NDWI thresholding formula:
        `(("Sentinel2_Green@1" - "Sentinel2_NIR@1") / ("Sentinel2_Green@1" + "Sentinel2_NIR@1")) >= 0.15`
        *(This computes the NDWI and sets cells with values $\ge 0.15$ to 1, and all others to 0).*
    
    *   **Output Layer:** Save as `water_mask.tif`.

---

### Step 2: Calculate Chlorophyll-a Indicator (NDCI Calculation)

*   **What We Are Doing:** Computing the Normalized Difference Chlorophyll Index (NDCI) over the water-masked pixels.
    
*   **Why This Step is Needed:** Chlorophyll-a absorbs strongly in the red spectrum (665 nm) and exhibits a peak reflectance jump in the red-edge spectrum (705 nm). The NDCI ratio isolates this response, providing a direct indicator of phytoplankton and algal bloom density.
    
*   *Input:* Sentinel-2 Red (B4) and Red-Edge (B5) bands, and the `water_mask.tif`.
    
*   *Output:* NDCI index grid (`reservoir_ndci.tif`).
    
*   *How to Fill the SAGA Form in QGIS:*
    
    *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Calculus** > **Grid Calculator**.
    
    *   **Grids:** Add `Sentinel2_Red_B4.tif` and `Sentinel2_RedEdge_B5.tif`.
    
    *   **Formula:** Enter the normalized difference formula:
        `(g2 - g1) / (g2 + g1)`
        *(where `g2` is the Red-Edge band B5 and `g1` is the Red band B4).*
    
    *   **Result:** Save the raw index grid. Run the Raster Calculator to multiply it by the `water_mask.tif` to set land areas to NoData:
        `"raw_ndci@1" / "water_mask@1"`
    
    *   **Final Output:** Save as `reservoir_ndci.tif`.

---

### Step 3: Calculate Total Suspended Solids (TSS Index)

*   **What We Are Doing:** Computing the Red-to-Green band ratio to highlight mineral turbidity.
    
*   **Why This Step is Needed:** Suspended mineral sediment and soil particles scatter red visible light back to the sensor, whereas clean water absorbs it. The Red/Green ratio isolates this scattering behavior from baseline water absorption.
    
*   *Input:* Sentinel-2 Red (B4) and Green (B3) bands, and the `water_mask.tif`.
    
*   *Output:* TSS index grid (`reservoir_tss_index.tif`).
    
*   *How to Fill the SAGA Form in QGIS:*
    
    *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Calculus** > **Grid Calculator**.
    
    *   **Grids:** Add `Sentinel2_Red_B4.tif` and `Sentinel2_Green_B3.tif`.
    
    *   **Formula:** Enter the ratio formula:
        `g1 / g2`
        *(where `g1` is the Red band B4 and `g2` is the Green band B3).*
    
    *   **Result:** Multiply the ratio output by the `water_mask.tif` using the Raster Calculator to remove terrestrial pixels:
        `"raw_tss_index@1" / "water_mask@1"`
    
    *   **Final Output:** Save as `reservoir_tss_index.tif`.

---

### Step 4: Extract Satellite Index Values at In-Situ Points

*   **What We Are Doing:** Sampling the values of the calculated NDCI and TSS index grids at the exact GPS coordinate locations of our water quality field stations.
    
*   **Why This Step is Needed:** To calibrate satellite indices to physical units ($mg/L$ or $\mu\text{g/L}$), we must pair ground-truth laboratory measurements with their corresponding satellite pixel values.
    
*   *Input:* Vector sampling stations layer (`sampling_stations.gpkg`) and index rasters (`reservoir_ndci.tif`, `reservoir_tss_index.tif`).
    
*   *Output:* Point vector layer updated with raster attribute columns (`ndci_val`, `tss_val`).
    
*   *How to Fill the SAGA Form in QGIS:*
    
    *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Shapes - Grid** > **Add Grid Values to Points**.
    
    *   **Grids:** Select `reservoir_ndci.tif` and `reservoir_tss_index.tif`.
    
    *   **Shapes:** Select `sampling_stations.gpkg`.
    
    *   **Interpolation:** Select `[0] Nearest Neighbor` *(this ensures the exact value of the intersecting pixel is sampled without interpolation smoothing).*
    
    *   **Result:** Save as a new layer `calibrated_station_points.gpkg`.

---

### Step 5: Empirical Regression & Concentration Calibration

*   **What We Are Doing:** Deriving calibration regression equations and applying them to the index rasters to generate physical concentration maps of TSS ($mg/L$) and Chlorophyll-a ($\mu\text{g/L}$).
    
*   **Why This Step is Needed:** Raw index rasters only contain decimal ratios. Applying the calibration equations converts these grids into actual concentration values that can be compared against water safety standards.
    
*   *Input:* Sampled attribute table of `calibrated_station_points.gpkg` and index rasters.
    
*   *Output:* Calibrated concentration rasters (`chlorophyll_conc.tif`, `tss_conc.tif`).
    
*   *Procedural Steps:*
    
    1.  **Extract the Calibration Table:** Open the attribute table of `calibrated_station_points.gpkg` and compile the matched values:
        
        | Station ID | Easting (m) | Northing (m) | Sampled NDCI ($X_1$) | Lab Chl-a ($\mu\text{g/L}$) ($Y_1$) | Sampled Red/Green ($X_2$) | Lab TSS ($mg/L$) ($Y_2$) |
        | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
        | **Station 1** | 345200 | 4531200 | 0.08 | 24.6 | 0.45 | 12.0 |
        | **Station 2** | 346100 | 4532100 | 0.22 | 46.2 | 0.85 | 32.5 |
        | **Station 3** | 344800 | 4530800 | 0.02 | 15.4 | 0.32 | 6.2 |
        | **Station 4** | 347000 | 4532800 | -0.05 | 4.6 | 0.22 | 3.1 |
        | **Station 5** | 345900 | 4531700 | 0.15 | 35.8 | 0.68 | 24.0 |
    
    2.  **Formulate Regression Equations:** Plot the variables in a spreadsheet and fit linear trendlines:
        *   **Chlorophyll-a Calibration Equation:**
            $$\text{Chlorophyll-a } (\mu\text{g/L}) = 154.2 \times \text{NDCI} + 12.3 \quad (R^2 = 0.96)$$
        *   **TSS Calibration Equation:**
            $$\text{TSS } (mg/L) = 46.8 \times \text{Red/Green Ratio} - 8.5 \quad (R^2 = 0.94)$$
    
    3.  **Run QGIS Raster Calculator Calibration:**
        *   Go to **Raster** > **Raster Calculator...**.
        *   Apply the Chlorophyll-a equation:
            `("reservoir_ndci@1" * 154.2) + 12.3`
        *   Save the output as `chlorophyll_conc.tif`.
        *   Run again applying the TSS equation:
            `("reservoir_tss_index@1" * 46.8) - 8.5`
        *   Save the output as `tss_conc.tif`.

---

## 4. Hydrological and Environmental Significance

*   **Eutrophication Monitoring:** High NDCI values ($>0.1$) and elevated chlorophyll concentrations ($>20\text{ }\mu\text{g/L}$) signify active algal blooms, indicating severe agricultural nutrient runoff or wastewater discharge.

*   **Water Treatment Optimization:** Real-time TSS and turbidity mapping helps municipal water utilities adjust chemical coagulant dosages at water treatment plant intakes, lowering treatment costs and preventing membrane clogging.

*   **Catchment Soil Erosion Auditing:** Analyzing spatial turbidity plumes in river networks following major storm events allows watershed managers to identify specific sub-catchments suffering from severe soil erosion and mass movements.
