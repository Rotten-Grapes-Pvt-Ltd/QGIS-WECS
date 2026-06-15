# Practical Session: Processing Satellite Data and Calculating Water Indices

This practical session guides you through downloading Sentinel-2 multispectral bands, generating color composites, clipping datasets to a watershed, and running index equations in the QGIS Raster Calculator.

---

## 1. Setting Up the Workspace

1. In your project directory, create a folder for Day 2 data:

   * `data/raster/sentinel2/`

   * `data/vector/watershed/`

2. Save your QGIS project as `Day2_Remote_Sensing.qgz` in the `projects/` folder.

3. Configure Project CRS to **WGS 84 / UTM Zone 45N (EPSG:32645)**.

---

## 2. Creating a False Color Composite (FCC)

1. Import Sentinel-2 Bands 2 (Blue), 3 (Green), 4 (Red), and 8 (NIR) into QGIS.

2. Build a virtual raster stack to combine these bands:

   * Go to **Raster** > **Miscellaneous** > **Build Virtual Raster**.

   * Click **Input layers** and select Bands 8, 4, and 3.

   * Check the box for **Place each input file into a separate band**.

   * Save the output as `sentinel2_fcc.vrt`.

3. In the Layers panel, right-click `sentinel2_fcc.vrt` and select **Properties** > **Symbology**.

   * Set **Red band** to `Band 1` (NIR).

   * Set **Green band** to `Band 2` (Red).

   * Set **Blue band** to `Band 3` (Green).

   * Click **Apply**.

4. Vegetation will display as bright red; water bodies will display as dark blue or black.

---

## 3. Clipping Rasters to Basin Boundaries

1. Load your watershed boundary vector file (e.g., `catchment.gpkg`).

2. Go to **Raster** > **Extraction** > **Clip Raster by Mask Layer**.

   * **Input Layer:** `sentinel2_fcc.vrt`.

   * **Mask Layer:** `catchment.gpkg`.

   * **Target CRS:** Match project CRS (`EPSG:32645`).

   * Save the output to `data/raster/catchment_satellite_clip.tif`.

---

## 4. Calculating NDWI in the Raster Calculator

1. Click **Raster** > **Raster Calculator**.

2. Double-click the Sentinel-2 bands in the bands list to construct the NDWI equation:
   $$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$
   Assuming Band 3 (Green) is `B3` and Band 8 (NIR) is `B8`, enter:
   `("B3@1" - "B8@1") / ("B3@1" + "B8@1")`

3. Save the output raster as `data/raster/catchment_ndwi.tif`.

4. Click **OK** to run the calculator. The output is a gray gradient where positive values represent water surfaces.

---

## 5. Generating a Binary Water Mask

1. Go to the **Processing Toolbox** and search for **Reclassify by Table**.

2. **Input Layer:** `catchment_ndwi.tif`.

3. Click **Reclassification table** `...` to define threshold ranges:

| Minimum Value | Maximum Value | Value |
| :--- | :--- | :--- |
| $-1.0$ | $0.0$ | $0$ (Non-water) |
| $0.0$ | $1.0$ | $1$ (Water) |

4. Click **OK** and save the output as `data/raster/binary_water_mask.tif`.

5. Run the tool. The output is a binary grid containing only $1	ext{s}$ (water) and $0	ext{s}$ (land).
