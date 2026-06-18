# Introduction to Time-Series Analysis

Time-series analysis in GIS involves analyzing a sequence of spatial datasets collected over time. By observing changes across multiple days, seasons, or decades, researchers can detect trends, monitor environmental degradation, and predict future states.

---

## 1. Multi-Temporal Datasets and Preprocessing

To perform reliable change detection, satellite imagery and grids must be preprocessed to ensure that differences observed on the ground represent actual landscape changes rather than artifacts of sensor angle, atmosphere, or offset coordinates.

*   **Sensor Consistency & Calibration:**
    
    Different satellite missions have minor variations in spectral band definitions (e.g., Landsat-8 vs. Landsat-9, or Sentinel-2A vs. Sentinel-2B).
    
    Analysts must calibrate Digital Numbers (DN) to radiance or Top-Of-Atmosphere (TOA) reflectance to enable comparison.

*   **Atmosphere Correction:**
    
    Scattering and absorption by atmospheric aerosols and gases distort the signal.
    
    Converting TOA reflectance to Bottom-Of-Atmosphere (BOA) reflectance—also known as Surface Reflectance (SR)—is required for time-series analysis.

*   **Geometric Registration:**
    
    Images must align perfectly pixel-to-pixel.
    
    Orthorectification uses Digital Elevation Models (DEMs) to correct for terrain displacement, preventing false changes along steep mountain ridges.

*   **Temporal Resolution:**
    
    The frequency of satellite revisits (e.g., 5 days for Sentinel-2, 16 days for Landsat) dictates the level of seasonal dynamics that can be captured.

---

## 2. Core Change Detection Concepts

Once data is preprocessed, several mathematical and spatial techniques are used to extract change areas.

*   **Post-Classification Comparison:**
    
    Each temporal image is classified independently into categories (e.g., water, forest, urban).
    
    The resulting thematic maps are compared using matrix analysis.
    
    *Advantage:* Identifies the specific nature of change (e.g., "Forest to Agriculture").
    
    *Disadvantage:* Errors in individual classifications multiply in the final change map.

*   **Raster Differencing (Image Subtraction):**
    
    Subtracts pixel values of a baseline raster from a comparison raster.
    
    Used for continuous datasets like elevation (DEM differencing to calculate erosion/deposition) or spectral indices.
    
    $$D = R_{\text{date2}} - R_{\text{date1}}$$

*   **Trend Analysis (Mann-Kendall and Sen's Slope):**
    
    Evaluates changes across a long, continuous series (e.g., 30 years of daily rainfall grids).
    
    The Mann-Kendall test detects if a trend is statistically significant.
    
    Sen's Slope determines the magnitude of the trend over time.

---

## 3. Step-by-Step Exercise: Change Detection via Raster Differencing

In this exercise, we will compute elevation changes in a river corridor before and after the monsoon season to map channel erosion and sedimentation.

1.  **Load Datasets:**
    
    Open QGIS. Load `dem_pre_monsoon.tif` and `dem_post_monsoon.tif` into the Layers Panel.

2.  **Open Raster Calculator:**
    
    Navigate to **Raster** > **Raster Calculator...**.

3.  **Calculate Difference:**
    
    Enter the following expression:
    
    `"dem_post_monsoon@1" - "dem_pre_monsoon@1"`
    
    Set the output layer name to `elevation_change.tif` and select your project directory. Click **OK**.

4.  **Style Output:**
    
    Open the Properties dialog for `elevation_change.tif` and select **Symbology**.
    
    Change the Render Type to **Singleband pseudocolor**.
    
    Set the Color Ramp to a diverging ramp (e.g., *RdBu* - Red to Blue).
    
    Set the Mode to **Continuous** or **Equal Interval**.
    
    Configure the color classes:
    
    *   Negative values (Red): Represents erosion (surface loss).
    
    *   Near-zero values (White): Represents stable ground.
    
    *   Positive values (Blue): Represents deposition/sedimentation (surface gain).

5.  **Calculate Change Statistics:**
    
    Go to **Processing Toolbox** > **Raster Analysis** > **Raster Layer Unique Values Report** or use **Zonal Statistics** to calculate the total area of erosion versus deposition within the river basin.
