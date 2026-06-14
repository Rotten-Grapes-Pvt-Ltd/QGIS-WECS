# Practical Session: Setting Up QGIS and Basic Data Operations

This practical session guides you through the process of setting up a QGIS project, organizing datasets, loading vector/raster layers, exploring attributes, and performing coordinate conversions.

---

## 1. Recommended Project Directory Structure
Before starting QGIS, always set up a structured directory on your computer to store your project files. This prevents broken file links during analysis.

```text
QGIS_Project_Root/
├── data/
│   ├── vector/      # Geopackages, shapefiles, GeoJSON
│   └── raster/      # DEMs, satellite imagery, land cover
├── projects/        # QGIS project files (.qgz)
└── outputs/         # Exported maps (PDF/PNG), report tables
```

---

## 2. Launching QGIS and Project Setup

1. Launch **QGIS Desktop**.

2. Click **Project** > **New** to create a clean project workspace.

3. Save your project immediately: Click **Project** > **Save As**, navigate to your `projects/` directory, and save it as `Day1_Introduction.qgz`.

4. Define Project Coordinate Reference System:

   * Look at the bottom-right corner of the QGIS interface. Click on the projection code (which defaults to `EPSG:4326` or a local zone).

   * In the search box, type `32645` (WGS 84 / UTM Zone 45N).

   * Select it and click **Apply** and **OK**. Your map canvas is now set to project in meters.

---

## 3. Loading Vector and Raster Datasets
You can load datasets using the **Browser Panel** or the **Layer Manager**:

* **Method A (Browser Panel):**

  1. In the Browser Panel, locate the directories where your training data is stored.

  2. Drag and drop vector layers (e.g., administrative boundaries) and raster layers (e.g., ALOS PALSAR DEM) directly onto the map canvas.

* **Method B (Layer Menu):**

  1. Click **Layer** > **Add Layer** > **Add Vector Layer**.

  2. Select your file (e.g., administrative districts GeoPackage) and click **Add**.

  3. Click **Layer** > **Add Layer** > **Add Raster Layer** to load your elevation GeoTIFF.

---

## 4. Basic Navigation and Map Controls
Use the toolbar at the top of the interface to navigate the map canvas:

* **Pan Tool (Hand Icon):** Drag to move across the map.

* **Zoom In/Out (Magnifying Glasses):** Click and drag a box to change map zoom levels.

* **Zoom Full (Three arrows pointing outwards):** Resizes the map zoom to show the full extent of all loaded layers.

* **Identify Features Tool (Info Icon):** Click on any vector boundary or raster pixel to view its attributes in a side panel.

---

## 5. Exploring Attribute Tables

1. In the **Layers Panel**, right-click on your vector layer (e.g., districts) and select **Open Attribute Table**.

2. Examine the rows (features) and columns (attribute fields).

3. **Filter features using expressions:**

   * Click the **Select Features Using an Expression** button (yellow square with an epsilon symbol $\varepsilon$).

   * Double-click **Fields and Values** in the middle list, and double-click `Elevation` (or the relevant field name).

   * Build the expression: `"Elevation" > 3000`

   * Click **Select Features** and close the dialog. The districts matching this query will be highlighted in yellow on the map canvas.

---

## 6. Layer Styling and Visualization

1. In the **Layers Panel**, right-click your raster DEM layer and select **Properties** > **Symbology**.

2. Change the **Render Type** from `Singleband gray` to `Singleband pseudocolor`.

3. Under **Color ramp**, select a color gradient (e.g., `Terrain` or `Spectral`).

4. Click **Classify** and then **Apply**. Your elevation grid will display as a colored altitude map.

---

## 7. Reprojecting and Saving a Vector Layer
To permanently reproject a layer to a different coordinate reference system:

1. Right-click on your vector layer in the Layers panel and select **Export** > **Save Features As**.

2. In the export dialog:

   * **Format:** Select `GeoPackage`.

   * **File Name:** Click `...` and save it to `data/vector/reprojected_districts.gpkg`.

   * **CRS:** Click the globe icon and search for `EPSG:32645` (WGS 84 / UTM Zone 45N).

3. Click **OK**. The new, reprojected layer will be saved and added to your map canvas.
