# Spectral Indices

Spectral indices combine reflectance values from multiple bands to isolate and highlight specific surface features.

---

## 1. Core Environmental Indices

* **NDVI (Normalized Difference Vegetation Index):**
  $$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$
  Measures vegetation density and chlorophyll activity.

* **NDWI (Normalized Difference Water Index):**
  $$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$
  Identifies open water features.

* **SAVI (Soil Adjusted Vegetation Index):**
  $$\text{SAVI} = \frac{(\text{NIR} - \text{Red}) \times (1 + L)}{\text{NIR} + \text{Red} + L}$$
  Where $L$ is a soil brightness correction factor ($0.5$ is standard). Used in areas with sparse vegetation where soil background reflections distort NDVI values.

* **NDBI (Normalized Difference Built-up Index):**
  $$\text{NDBI} = \frac{\text{SWIR} - \text{NIR}}{\text{SWIR} + \text{NIR}}$$
  Highlights urban and built-up areas.
