# Spectral Indices

Spectral indices combine reflectance values from multiple spectral bands to highlight specific physical surface properties, such as vegetation health, open water, soil characteristics, or urban footprints.

---

## 1. The Physics of Spectral Refectance

Different earth features interact uniquely with electromagnetic radiation:

```text
  SPECTRAL SIGNATURE COMPARISON
  Reflectance %
   100 |
       |                           --. (Vegetation - NIR peak)
    50 |                 _..------'
       |       .--------'             `--. (Bare Soil - Gradual rise)
       |      /
     0 +-----+---------+---------+---------+
            Blue     Green      Red       NIR
```

*   **Vegetation:** Strongly absorbs Red light (chlorophyll absorption) and strongly reflects Near-Infrared (NIR) energy (spongy mesophyll cell structure).

*   **Water:** Absorbs almost all NIR and Shortwave Infrared (SWIR) energy, showing moderate reflectance only in the Blue and Green bands.

*   **Built-up / Urban:** Shows a gradual increase in reflectance from visible to SWIR bands, often peaking in the SWIR range.

---

## 2. Core Environmental Indices

By calculation ratios of these bands, we can minimize topographic shadow effects and atmospheric variations.

### Normalized Difference Vegetation Index (NDVI)

NDVI is the standard index for monitoring vegetation density, health, and phenology.

$$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$

*   **Physical Meaning:** Values range from $-1.0$ to $+1.0$.
    
    *   **$[-1.0 \text{ to } 0.0]$:** Water bodies, snow, and clouds.
    
    *   **$[0.0 \text{ to } 0.15]$:** Bare soil, sand, and rocks.
    
    *   **$[0.15 \text{ to } 0.4]$:** Sparse vegetation, grasslands, and crops in early growth stages.
    
    *   **$[0.4 \text{ to } 1.0]$:** Dense canopy forests and healthy active crops.

### Soil Adjusted Vegetation Index (SAVI)

SAVI introduces a soil brightness correction factor ($L$) to reduce background soil noise in regions with sparse vegetation.

$$\text{SAVI} = \frac{(\text{NIR} - \text{Red}) \times (1 + L)}{\text{NIR} + \text{Red} + L}$$

*   **Parameter $L$:** Ranges from $0.0$ (very dense vegetation) to $1.0$ (barren soil).
    
    The standard value of $L = 0.5$ is applied in most agricultural settings.

### Normalized Difference Water Index (NDWI)

NDWI is used to delineate open surface water features and monitor flood extent.

$$\text{NDWI}_{\text{McFeeters}} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$

*   **Water Contrast:** Since water absorbs NIR and reflects Green, water bodies yield positive values, while soil and vegetation yield negative values.

### Normalized Difference Built-up Index (NDBI)

NDBI is designed to map urban footprints, buildings, and concrete structures.

$$\text{NDBI} = \frac{\text{SWIR} - \text{NIR}}{\text{SWIR} + \text{NIR}}$$

*   **Urban Contrast:** Concrete and asphalt reflect SWIR more strongly than NIR, yielding positive values for built-up surfaces.

---

## 3. Step-by-Step Exercise: Computing NDVI and NDWI in QGIS

1.  **Import Band Rasters:**
    
    Load `band_4_red.tif` and `band_8_nir.tif` (from a Sentinel-2 scene) into QGIS.

2.  **Open Raster Calculator:**
    
    Go to **Raster** > **Raster Calculator...**.

3.  **Write NDVI Expression:**
    
    Enter the formula using the layer names in your panel:
    
    `("band_8_nir@1" - "band_4_red@1") / ("band_8_nir@1" + "band_4_red@1")`
    
    *Note:* Ensure the output layer name is specified as `ndvi_output.tif` and the output format is GeoTIFF.

4.  **Review the Output:**
    
    Click **OK**.
    
    Double-click `ndvi_output.tif` in the Layers Panel.
    
    Go to **Symbology** and set Render Type to **Singleband pseudocolor**.
    
    Select a green color ramp (e.g., *YlGn* - Yellow to Green) to visualize the vegetation distribution.

5.  **Compute NDWI:**
    
    Load `band_3_green.tif`.
    
    Re-open the Raster Calculator and run:
    
    `("band_3_green@1" - "band_8_nir@1") / ("band_3_green@1" + "band_8_nir@1")`
    
    Save as `ndwi_output.tif`.
    
    Style the output with a blue ramp.
    
    Apply a threshold filter to keep only values $> 0$ to cleanly isolate the water bodies.
