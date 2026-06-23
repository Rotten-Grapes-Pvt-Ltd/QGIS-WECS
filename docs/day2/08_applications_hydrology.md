# Remote Sensing Applications in Hydrology

By combining and calculating spectral bands, we can extract physical parameters directly from satellite images. This section explains the formulas and workflows for mapping surface water, snow extent, and vegetation health.

---

## 1. Surface Water Monitoring (NDWI vs. MNDWI)

Water indexing uses the differences in reflectance between visible and infrared bands to extract water boundaries.

### Normalized Difference Water Index (NDWI)
The NDWI (developed by McFeeters) maps open water bodies by leveraging the fact that water has high reflectance in green light and high absorption in Near-Infrared (NIR) light:

$$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$

* **Sentinel-2 Bands:**
  $$\text{NDWI} = \frac{\text{Band 3} - \text{Band 8}}{\text{Band 3} + \text{Band 8}}$$

* **Landsat 8/9 Bands:**
  $$\text{NDWI} = \frac{\text{Band 3} - \text{Band 5}}{\text{Band 3} + \text{Band 5}}$$

---

### Modified Normalized Difference Water Index (MNDWI)
In watersheds with built-up areas (such as concrete, asphalt, and rooftops), standard NDWI often misclassifies urban surfaces as water because built-up features can yield positive NDWI values. The MNDWI (developed by Xu) solves this by replacing the NIR band with a Shortwave Infrared (SWIR) band:

$$\text{MNDWI} = \frac{\text{Green} - \text{SWIR}}{\text{Green} + \text{SWIR}}$$

* **Sentinel-2 Bands:**
  $$\text{MNDWI} = \frac{\text{Band 3} - \text{Band 11}}{\text{Band 3} + \text{Band 11}}$$

* **Landsat 8/9 Bands:**
  $$\text{MNDWI} = \frac{\text{Band 3} - \text{Band 6}}{\text{Band 3} + \text{Band 6}}$$

* **Urban Noise Suppression:** Built-up features reflect SWIR energy much more strongly than NIR energy. Using SWIR in the denominator drives the MNDWI index value strongly negative for urban features, while water remains strongly positive. This suppresses urban noise, making MNDWI the standard index for mapping water bodies near populated areas.

* **Thresholding:** While a value of $0$ is the theoretical threshold separating water ($&gt;0$) from land ($\le0$), wet soils or shadows can push land values positive. Hydrologists inspect the local index histogram to adjust the threshold manually (e.g., setting the water limit to $0.1$ or $-0.05$ to optimize the boundary).

---

## 2. Flood Mapping using SAR Backscatter

Active Synthetic Aperture Radar (SAR) sensors emit microwave pulses and measure the strength of the backscattered signal ($\sigma^0$) returned to the sensor, measured in decibels (dB):

* **Calm Water (Specular Reflection):** Calm water acts as a flat mirror. The radar pulse bounces away from the satellite, returning very little energy. Water appears **pitch black** in SAR images, with typical backscatter values below $-16\text{ dB}$.

* **Dry Land (Diffuse Scattering):** Land surfaces, vegetation, and gravel scatter the radar signal in multiple directions. A portion of the energy returns to the satellite, appearing as **gray**.

---

### Sentinel-1 GRD Preprocessing Workflow
To extract floodwater boundaries from raw Sentinel-1 Ground Range Detected (GRD) data, the raster must undergo a calibration pipeline:

1. **Apply Orbit File:** Corrects satellite position parameters using precise orbital data to ensure spatial accuracy.

2. **Thermal Noise Removal:** Minimizes background system noise, especially in cross-polarized channels (VH).

3. **Radiometric Calibration:** Converts pixel values into physical backscatter coefficient ($\sigma^0$) values.

4. **Terrain Correction (Orthorectification):** Uses a DEM (like Copernicus GLO-30) to correct geometric layover and shadow distortions caused by rugged terrain.

5. **Decibel Conversion:** Transforms the linear backscatter values into a logarithmic scale:
   $$\sigma^0\text{ (dB)} = 10 \log_{10} \sigma^0$$

---

### Bimodal Histogram Segmentation
Plotting a histogram of the calibrated backscatter values of a flooded area typically reveals a **bimodal distribution** (two peaks):

* **First Peak (Low dB, e.g., $-22\text{ dB}$):** Represents smooth surfaces (floodwater).

* **Second Peak (High dB, e.g., $-10\text{ dB}$):** Represents rougher dry land.

* **Threshold Selection:** The valley point between these two peaks defines the threshold value (usually between $-15\text{ dB}$ and $-18\text{ dB}$ for VH polarization). All pixels with backscatter values below this threshold are classified as floodwater.

* **Slope Masking:** Steep mountain slopes facing away from the sensor receive no radar energy (radar shadows), which appear black and mimic calm water. Hydrologists use a DEM to generate a slope mask, filtering out all identified water pixels on slopes steeper than $5^{\circ}$ to eliminate mountain shadow false positives.

---

## 3. Vegetation Monitoring & Crop Coefficients (NDVI)

The Normalized Difference Vegetation Index (NDVI) measures vegetation density and health by comparing chlorophyll absorption in red light and cellular scattering in NIR light:

$$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$

* **Sentinel-2 Bands:**
  $$\text{NDVI} = \frac{\text{Band 8} - \text{Band 4}}{\text{Band 8} + \text{Band 4}}$$

* **Hydrological Role:** NDVI values correlate with **Leaf Area Index (LAI)** and **Crop Coefficients ($K_c$)**. Water resource planners use crop coefficients derived from NDVI time-series to calculate agricultural evapotranspiration (ET) and optimize irrigation water allocations.
  
  * *Bare soil:* $0$ to $0.2$
  
  * *Sparse/Stressed vegetation:* $0.2$ to $0.5$
  
  * *Dense, healthy canopy:* $0.6$ to $1.0$

---

## 4. Snow Cover Mapping & SCA (NDSI)

The Normalized Difference Snow Index (NDSI) distinguishes snow from clouds. While both snow and clouds appear bright white in visible bands, snow absorbs SWIR light whereas clouds reflect it:

$$\text{NDSI} = \frac{\text{Green} - \text{SWIR}}{\text{Green} + \text{SWIR}}$$

* **Sentinel-2 Bands:**
  $$\text{NDSI} = \frac{\text{Band 3} - \text{Band 11}}{\text{Band 3} + \text{Band 11}}$$

* **Hydrological Role:** Pixels with $\text{NDSI} &gt; 0.4$ are typically classified as snow cover. Delineating the **Snow Cover Area (SCA)** over time allows hydrologists to estimate snowmelt runoff volumes, which are critical for predicting seasonal flows in rivers fed by Himalayan snowmelt.

---

## 5. Guided Class Exercises

### Exercise 1: Selecting NDWI vs. MNDWI for an urban-adjacent reservoir
An analyst is tasked with mapping the surface water boundary of a municipal drinking water reservoir located adjacent to a growing town. 

They calculate standard NDWI using Green and NIR bands. On the final map, several residential blocks and commercial shopping centers are highlighted as "water," distorting the calculated surface area of the reservoir.

1. Explain the spectral reason why standard NDWI misclassified these urban areas as water.

2. Recommend an alternative spectral index to fix this, and explain how it suppresses the urban built-up noise.

??? check "Answer Key - Exercise 1"

    1. **Cause of NDWI Urban Misclassification:**
    
        * Built-up surfaces (like concrete, metal roofs, and asphalt) exhibit a spectral response where reflectance in the green band is relatively high, and reflectance in the NIR band is low-to-moderate. 
        
        * When processed through the NDWI formula, the low NIR value results in a positive index score ($&gt;0$). Because the water threshold is set at $0$, these built-up areas are classified as water.
        
    2. **Recommended Index and Suppression Mechanism:**
    
        * The analyst should use the **Modified Normalized Difference Water Index (MNDWI)**, which replaces the NIR band with a SWIR band:
        
          $$\text{MNDWI} = \frac{\text{Green} - \text{SWIR}}{\text{Green} + \text{SWIR}}$$
          
        * Concrete and urban materials reflect SWIR radiation very strongly. Because the SWIR value in the formula is much larger than the Green value, subtracting it in the numerator forces the MNDWI value strongly negative ($&lt;0$). 
        
        * Water continues to absorb SWIR energy completely, keeping its MNDWI value strongly positive. This separates water ($&gt;0$) from urban surfaces ($&lt;0$), eliminating the built-up noise.

---

### Exercise 2: Formulating a SAR flood boundary thresholding workflow
A hydrologist is mapping a severe monsoon flood in the low-lying Terai plains of Nepal using Sentinel-1 SAR imagery. The study area lies at the base of the Siwalik hills. 

1. Outline the step-by-step calibration and preprocessing sequence required to convert raw Sentinel-1 GRD data into a format suitable for flood thresholding.

2. Explain how to select the backscatter threshold value using the bimodal histogram method.

3. The raw threshold output maps several false flood zones along the steep slopes of the Siwalik hills. Explain the cause of these errors and how to remove them using a secondary terrain dataset.

??? check "Answer Key - Exercise 2"

    1. **Preprocessing Workflow:**
    
        * **Apply Orbit File:** Corrects orbit positioning.
        
        * **Thermal Noise Removal:** Eliminates background cross-polarized signal noise.
        
        * **Radiometric Calibration:** Converts pixel counts to radar backscatter coefficient ($\sigma^0$).
        
        * **Terrain Correction (Orthorectification):** Corrects layover and geometric slope distortions using a DEM.
        
        * **Decibel Conversion:** Transforms $\sigma^0$ to decibel units ($\text{dB}$) via $10 \log_{10} \sigma^0$.
        
    2. **Threshold Selection via Bimodal Histogram:**
    
        * Plot the pixel histogram of the processed dB band. Identify the two peaks: the left peak represents low backscatter (smooth floodwater), and the right peak represents higher backscatter (rough land).
        
        * Locate the minimum point (valley) between these two peaks. This value (typically around $-16\text{ dB}$ on the VH channel) is the threshold. Pixels below this value are classified as water.
        
    3. **slope Filtering for Mountain Shadows:**
    
        * **Cause of Error:** The steep slopes of the Siwalik hills facing away from the radar sensor receive no signal (radar shadows). These areas return zero signal, appearing pitch black (low dB values) and mimicking calm floodwater.
        
        * **Resolution:** Derive a slope raster in degrees from a DEM. In the Raster Calculator, apply a slope threshold mask (e.g., `Slope <= 5 degrees`). Multiply the flood raster by this mask. This deletes all classified flood pixels located on slopes steeper than $5^{\circ}$ (where standing water cannot physically accumulate), successfully removing the false mountain shadow flood zones.
