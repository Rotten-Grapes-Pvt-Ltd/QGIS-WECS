# Satellite Bands, Composites, and Spectral Indices

Multispectral satellites record the Earth's surface across multiple parts of the electromagnetic spectrum. By combining these bands into color composites or computing mathematical ratios (spectral indices), hydrologists can extract surface water boundaries, map soil moisture, monitor vegetation health, and delineate snow cover.

!!! tip "Presentation Slides"
    You can download or view the lecture slides for this topic: [Bands_Composites_Indices.pdf](presentations/04_Bands_Composites_Indices.pdf)

---

## 1. How Color Composites Work
Computer screens display color images by combining three primary color channels: **Red (R)**, **Green (G)**, and **Blue (B)**. A digital satellite image is a multi-layer stack of rasters, where each layer corresponds to a specific wavelength range. To create a color image, we assign three of these spectral bands to the screen's RGB channels:

```text
    Multispectral Image Bands                  Computer Screen Channels
    +-----------------------+                 +-------------------------+
    |   Band X (e.g., NIR)  | --------------> |   Red Display Channel   |
    +-----------------------+                 +-------------------------+
    |   Band Y (e.g., Red)  | --------------> |  Green Display Channel  |
    +-----------------------+                 +-------------------------+
    |  Band Z (e.g., Green) | --------------> |   Blue Display Channel  |
    +-----------------------+                 +-------------------------+
```

By varying which band is mapped to which display channel, we highlight different physical properties of the landscape.

---

## 2. Common Band Composites in Hydrology

### 1. True Color (Natural Color)

* **Wavelengths mapped:** Red $\rightarrow$ Red | Green $\rightarrow$ Green | Blue $\rightarrow$ Blue.

* **Band Configurations:**

    * *Sentinel-2:* `B04, B03, B02`

    * *Landsat 8/9:* `B04, B03, B02`

* **Hydrological Application:** Mimics human vision. It is the best composite for identifying water turbidity, sediment plumes in reservoirs, and mapping river plume distribution in coastal zones.

### 2. Standard False Color (Vegetation FCC)

* **Wavelengths mapped:** Near-Infrared (NIR) $\rightarrow$ Red | Red $\rightarrow$ Green | Green $\rightarrow$ Blue.

* **Band Configurations:**

    * *Sentinel-2:* `B08, B04, B03`

    * *Landsat 8/9:* `B05, B04, B03`

* **Hydrological Application:** Vegetation reflects NIR light strongly due to cellular structure, so healthy canopy forest appears bright red. Since water absorbs NIR completely, open water bodies stand out as dark blue or solid black, making this composite excellent for delineating shoreline boundaries.

### 3. SWIR Agriculture & Water Composite

* **Wavelengths mapped:** Shortwave-Infrared (SWIR-1) $\rightarrow$ Red | Near-Infrared (NIR) $\rightarrow$ Green | Red $\rightarrow$ Blue.

* **Band Configurations:**

    * *Sentinel-2:* `B11, B08, B04`

    * *Landsat 8/9:* `B06, B05, B04`

* **Hydrological Application:** This composite maximizes the contrast between water and land. Water absorbs both NIR and SWIR, appearing deep dark blue or black. Healthy crops appear bright green, bare soils appear in shades of brown/orange, and built-up urban structures appear in magenta. It is the standard composite for mapping flooded fields and reservoir margins.

### 4. Atmospheric Penetration / Geology Composite

* **Wavelengths mapped:** SWIR-2 $\rightarrow$ Red | SWIR-1 $\rightarrow$ Green | NIR $\rightarrow$ Blue.

* **Band Configurations:**

    * *Sentinel-2:* `B12, B11, B08`

    * *Landsat 8/9:* `B07, B06, B05`

* **Hydrological Application:** Uses longer wavelengths that easily penetrate atmospheric haze, smoke, and thin clouds. Highly useful for mapping geological faults, drainage paths, or post-fire burn severity in mountainous catchments.

---

## 3. Land Surface Feature Appearance Matrix

| Composite Type | Open Water | Turbid/Sediment Water | Active Forest | Bare Soil / Sand | Built-Up / Urban | Snow / Ice |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **True Color (4,3,2)** | Deep Blue/Black | Light Green/Brown | Dark Green | Light Brown/White | Light Gray | White |
| **Vegetation FCC (8,4,3)** | Black/Dark Blue | Light Blue/Green | Bright Red | Gray/Brown | Cyan/Blue | White |
| **SWIR Agriculture (11,8,4)** | Solid Black | Dark Blue | Bright Green | Orange/Brown | Magenta/Pink | Bright Cyan |

---

## 4. Spectral Indices: Normalized Ratios
Single bands are sensitive to changes in sun angle, shadows, and topography. **Spectral Indices** solve this by calculating normalized ratios between two bands, isolating specific physical features while minimizing noise:

### 1. NDVI (Normalized Difference Vegetation Index)
NDVI measures chlorophyll activity. In hydrology, it is used to estimate forest canopy interception and scale evapotranspiration rates:

$$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}} \quad \text{or} \quad \text{NDVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04}} \quad \text{(Sentinel-2)}$$

* **Values:** Ranges from $-1.0$ to $+1.0$. Water and snow return negative values; bare soils return values close to zero ($0.0$ to $0.1$); dense forest canopies yield values close to $+1.0$.

### 2. NDWI (Normalized Difference Water Index - McFeeters)
Specifically formulated to highlight open water surfaces:

$$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}} \quad \text{or} \quad \text{NDWI} = \frac{\text{B03} - \text{B08}}{\text{B03} + \text{B08}} \quad \text{(Sentinel-2)}$$

* **Values:** Water features return positive values ($> 0.0$), while soil and vegetation yield negative or zero values.

* **Limitation:** Built-up urban areas reflect green and NIR light similarly to water, which can create false-positive water classifications in cities.

### 3. MNDWI (Modified Normalized Difference Water Index - Xu)
Replaces the Near-Infrared band with Shortwave-Infrared (SWIR) to solve the urban built-up noise issue:

$$\text{MNDWI} = \frac{\text{Green} - \text{SWIR-1}}{\text{Green} + \text{SWIR-1}} \quad \text{or} \quad \text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}} \quad \text{(Sentinel-2)}$$

* **Why it works:** SWIR absorbs water even more strongly than NIR. Urban surfaces reflect SWIR strongly, yielding strongly negative MNDWI values. This suppresses urban noise, making MNDWI the standard index for delineating lakes and rivers.

### 4. NDSI (Normalized Difference Snow Index)
Used to separate snow cover from clouds:

$$\text{NDSI} = \frac{\text{Green} - \text{SWIR-1}}{\text{Green} + \text{SWIR-1}} \quad \text{or} \quad \text{NDSI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}} \quad \text{(Sentinel-2)}$$

* **Why it works:** Snow is highly reflective in the visible green band but highly absorptive in the SWIR band. Clouds are highly reflective in both green and SWIR. Thus, snow returns high positive NDSI values, while clouds yield values close to zero.

---

## 5. Practical QGIS Workflow: Building Composites 
To build a custom composite from individual band files (e.g., bands downloaded from Landsat or Sentinel):

1. **Import Bands:** Load the required single-band GeoTIFFs into the QGIS Layers Panel.

2. **Build Virtual Raster:**

    * Navigate to **Raster** > **Miscellaneous** > **Build Virtual Raster**.

    * **Input Layers:** Select the individual band files in order (e.g., Red, Green, Blue, NIR, SWIR).

    * **Place each input file into a separate band:** **Check this option** to generate a multiband file.

    * Click **Run**. QGIS creates a temporary `Virtual` layer.

3. **Configure Symbology:**

    * Right-click the `Virtual` layer and select **Properties** > **Symbology**.

    * **Render Type:** Select **Multiband Color**.

    * **Red / Green / Blue Band mappings:** Assign your desired bands to the display channels (e.g., for Sentinel-2 SWIR Agriculture composite: Red channel = Band 5 (SWIR-1), Green channel = Band 4 (NIR), Blue channel = Band 3 (Red)).

    * Click **Apply**.

---

## 6. Review & Interactive Discussion Prompts (10 Minutes)

1. **Index Selection Trade-offs:**

    * Why would you prefer **MNDWI** over **NDWI** when mapping a river that passes through a major city?

    * If you are mapping snow cover in a catchment that is 80% covered by clouds, why is a simple threshold on the true-color band insufficient?

??? check "Answer Key - Discussion Prompt 1"

    * **MNDWI vs. NDWI in urban settings:**

        * Built-up concrete structures and buildings reflect green and NIR light similarly, yielding NDWI values near or above zero, which creates false-positive water classifications in cities.

        * MNDWI replaces the NIR band with SWIR-1. Because urban features reflect SWIR-1 energy strongly, MNDWI yields strongly negative values for built-up areas, suppressing urban noise and clearly isolating river networks.

    * **Why true-color fails for snow vs. clouds:**

        * Both snow and clouds reflect visible wavelengths strongly and appear bright white in a true-color (RGB 4,3,2) image, making them visually indistinguishable.

        * NDSI uses the SWIR-1 band. Because snow absorbs SWIR-1 energy while clouds reflect it, NDSI yields positive values for snow and values near zero for clouds, allowing clear separation.

2. **Band Combinations for Flood Response:**

    * During active flood events, skies are often overcast. What composite or sensor system (hint: SAR vs. Optical) would you select to capture flood extents under heavy cloud cover, and why?

??? check "Answer Key - Discussion Prompt 2"

    * **Sensor selection under cloud cover:**

        * Active Radar (SAR / Synthetic Aperture Radar) like the Sentinel-1 C-band should be selected.

        * SAR sensors emit their own microwave energy at wavelengths that easily penetrate clouds, fog, and rain, allowing flood boundaries to be mapped day or night regardless of weather conditions. Optical sensors (Landsat, Sentinel-2) are completely blocked by overcast skies during flood events.

3. **The Shadow Problem:**

    * Look back at the Land Surface Feature Appearance Matrix. Note how both deep open water and steep terrain shadows can appear black in a Vegetation FCC. Suggest a workflow combining spectral indices and digital elevation thresholds to isolate the true water body.

??? check "Answer Key - Discussion Prompt 3"

    * **Isolating water bodies from terrain shadows:**

        * **Step 1:** Calculate **MNDWI**. Open water bodies will return high positive values, while shadows cast on vegetated slopes or bare rock will yield negative or near-zero values.

        * **Step 2:** Calculate a **Slope grid** from a Digital Elevation Model (DEM). Water bodies are flat (slope $\approx 0^\circ$), whereas mountain faces casting steep shadows exhibit high slope angles.

        * **Step 3:** Apply a raster math threshold, mapping as water only those pixels that have both high MNDWI (e.g., $> 0.1$) and low slope values (e.g., $< 3^\circ$). This masks out mountain shadows.
