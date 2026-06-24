# Water Quality Assessment

Satellite remote sensing enables the continuous monitoring of water quality parameters across lakes, reservoirs, and rivers. By analyzing the optical reflectance signatures of water bodies, GIS specialists can map turbidity, Total Suspended Solids (TSS), and chlorophyll-a concentrations to identify pollution and algal blooms.

---

## 1. Core Objectives

*   **Delineate** water bodies using spectral masking.

*   **Map** turbidity and suspended sediment distribution.

*   **Monitor** chlorophyll-a concentrations to track eutrophication and algal blooms.

*   **Integrate** satellite maps with in-situ laboratory water quality samples.

---

## 2. Key GIS Inputs

*   **High-Resolution Multispectral Imagery:** Sentinel-2 (ideal due to its dedicated Red-Edge bands) or Landsat 8/9.
    
    *   *Atmospheric Correction:* Imagery must be processed to **Bottom-of-Atmosphere (BOA) reflectance** (using Sen2Cor or LaSRC) to remove atmospheric scattering noise.

*   **In-Situ Water Samples:** Coordinate point data containing laboratory measurements of Turbidity (NTU), TSS (mg/L), and Chlorophyll-a ($\mu\text{g/L}$).

---

## 3. Optical Properties of Water

Water features unique spectral signatures based on its constituents:

*   **Clear Water:** Absorbs almost all incoming light in the Near-Infrared (NIR) and Shortwave Infrared (SWIR) wavelengths. It reflects small amounts of blue-green visible light.

*   **Turbid Water / Sediment:** Suspended mineral sediment scatters light, increasing reflectance in the visible spectrum (specifically Red and Green bands).

*   **Algae / Chlorophyll-a:** Phytoplankton contain chlorophyll-a, which absorbs Red and Blue light, and reflects Green light. Dense algal blooms exhibit a strong "red edge" reflection jump in the NIR band, mimicking land vegetation.

---

## 4. Water Quality Indices & Empirical Formulas

To estimate quality parameters, spectral bands are processed using the Raster Calculator:

### Total Suspended Solids (TSS) & Turbidity

A widely used index for mapping suspended sediments utilizes the Red band or the Red/Green ratio:

$$\text{Sediment Index} = \frac{\text{Red}}{\text{Green}} \quad \text{or} \quad \text{TSS} = A \cdot e^{B \cdot \text{Red}}$$

Where $A$ and $B$ are calibration coefficients derived by regressing satellite reflectance against in-situ laboratory water samples.

### Chlorophyll-a & Algal Blooms (NDCI)

The **Normalized Difference Chlorophyll Index (NDCI)** utilizes Sentinel-2's Red-Edge band (Band 5, $705\text{ nm}$) and Red band (Band 4, $665\text{ nm}$) to isolate chlorophyll absorption:

$$NDCI = \frac{\text{Band 5} - \text{Band 4}}{\text{Band 5} + \text{Band 4}}$$

*   *NDCI Logic:* The Red band represents maximum chlorophyll absorption, while the Red-Edge band represents the reflection jump.

*   *Classification:* NDCI values $> 0.1$ represent active algal blooms and high eutrophication levels.

---

## 5. GIS Processing Workflow

1.  **Mask Land and Clouds:** Apply a water mask (e.g., NDWI $> 0.15$) to set all land, forest, and urban pixels to NoData.

2.  **Run Index Calculators:** Run the NDCI or TSS index formula in the **Raster Calculator** on the water-only pixels.

3.  **Perform Empirical Calibration:**
    
    *   Use the **Sample Raster Values** tool to extract satellite index values at the exact coordinate locations of in-situ sampling stations.
    
    *   Plot a scatterplot in Excel or Python comparing the extracted index values (X-axis) with laboratory-tested TSS or Chlorophyll concentrations (Y-axis).
    
    *   Derive the regression equation (e.g., $Y = 154.2 \cdot X + 12.3$) and apply it back to the index raster in QGIS to generate a calibrated concentration map.

---

## 6. Hydrological Significance

*   **Drinking Water Management:** Real-time turbidity mapping assists water utility operators in adjusting chemical treatment dosages at intake sites.

*   **Ecosystem Monitoring:** Early detection of toxic blue-green algal blooms (cyanobacteria) allows resource managers to close recreational lakes and issue public health alerts.

*   **Catchment Erosion Feedback:** Elevated turbidity levels in river systems during storm events point to severe soil erosion and landslide activities in upstream sub-catchments.


## 7. Data Sources & Acquisition

If you do not have satellite or ground sampling databases to monitor water quality:

*   **Sentinel-2 Level-2A (BOA) Reflectance:** Mandatory for index calculation. Download pre-processed Bottom-of-Atmosphere (BOA) tiles from the [Copernicus Browser](https://dataspace.copernicus.eu/). Select clear, cloud-free days during key agricultural or algal season phases.

*   **Landsat 8/9 Level-2 Surface Reflectance:** Download from [USGS EarthExplorer](https://earthexplorer.usgs.gov/).

*   **UNEP GEMStat (Global Water Quality Database):** Sourced from the United Nations Environment Programme, this database offers global in-situ freshwater quality data. Request historical dataset access at the [GEMStat Portal](https://gemstat.org/).
