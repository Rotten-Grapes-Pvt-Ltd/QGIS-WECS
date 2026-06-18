# Remote Sensing Applications in Hydrology

By combining and calculating spectral bands, we can extract physical parameters directly from satellite images. This section explains the formulas and workflows for mapping surface water, snow extent, and vegetation health.

---

## 1. Surface Water Monitoring (NDWI)
The **Normalized Difference Water Index (NDWI)** is used to extract open water features. It leverages the fact that water reflects green light but absorbs Near-Infrared (NIR) light:

$$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$

* **Sentinel-2 Bands:**
  $$\text{NDWI} = \frac{\text{Band 3} - \text{Band 8}}{\text{Band 3} + \text{Band 8}}$$

* **Landsat 8 Bands:**
  $$\text{NDWI} = \frac{\text{Band 3} - \text{Band 5}}{\text{Band 3} + \text{Band 5}}$$

* **Interpretation:**

  * $\text{NDWI} > 0$: Indicates water surfaces (rivers, lakes, reservoirs).

  * $\text{NDWI} \le 0$: Non-water surfaces (land, vegetation, clouds).

---

## 2. Flood Mapping using SAR Backscatter
Synthetic Aperture Radar (SAR) sensors emit microwave pulses and measure the strength of the returned signal (backscatter coefficient).

* **Calm Water:** Acts as a specular reflector, bouncing the signal away from the sensor. It returns very little energy to the satellite, appearing dark.

* **Dry Land:** Generates diffuse scattering, returning more energy and appearing bright.

* **Flood Delineation Workflow:**

  1. Acquire a Sentinel-1 SAR image collected during the flood event.

  2. Apply a log-transform to calculate backscatter values in decibels (dB).

  3. Inspect the histogram to identify a threshold value (typically between $-15\text{ dB}$ and $-18\text{ dB}$ for VH polarization).

  4. Classify all pixels with backscatter values below this threshold as floodwater.

---

## 3. Vegetation Monitoring (NDVI)
The **Normalized Difference Vegetation Index (NDVI)** measures vegetation density and health:

$$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$

* **Sentinel-2 Bands:**
  $$\text{NDVI} = \frac{\text{Band 8} - \text{Band 4}}{\text{Band 8} + \text{Band 4}}$$

* **Interpretation:** Healthy vegetation absorbs red light (for photosynthesis) and reflects NIR light, resulting in high NDVI values ($0.6$ to $0.9$). Bare soil and water yield low or negative NDVI values.

---

## 4. Snow Cover Mapping (NDSI)
The **Normalized Difference Snow Index (NDSI)** distinguishes snow from clouds. Snow reflects visible light but absorbs SWIR light, whereas clouds reflect both:

$$\text{NDSI} = \frac{\text{Green} - \text{SWIR}}{\text{Green} + \text{SWIR}}$$

* **Sentinel-2 Bands:**
  $$\text{NDSI} = \frac{\text{Band 3} - \text{Band 11}}{\text{Band 3} + \text{Band 11}}$$

* **Interpretation:** $\text{NDSI} > 0.4$ is typically classified as snow cover.
