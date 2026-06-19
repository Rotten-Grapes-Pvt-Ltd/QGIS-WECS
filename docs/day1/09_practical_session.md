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

---

## 8. Guided Mini-Assignment: Delineating a Watershed Layout Map

In this assignment, you will apply the skills learned during Day 1 to set up a structured GIS workspace, import datasets, perform coordinate transformations, and compile a formatted map layout for submission.

### Objective
Produce a georeferenced, styled watershed overview map for a selected catchment in Nepal, formatted on an $A4$ sheet, and export the output as a high-quality PDF.

### Provided Datasets
* `/data/vector/nepal_districts.gpkg` (Administrative divisions of Nepal, EPSG:4326).

* `/data/vector/major_rivers.gpkg` (Stream centerlines, EPSG:4326).

* `/data/raster/basin_dem.tif` (ALOS PALSAR Digital Elevation Model, EPSG:4326).

### Step-by-Step Requirements

#### Step 1: Workspace Organization
Create a directory structure on your computer named `WECS_Assignment_Day1` containing subdirectories for `data`, `projects`, and `outputs`. Copy the provided datasets into their respective folders.

#### Step 2: Project CRS Setup
Open QGIS, create a new project, and configure the project CRS to **WGS 84 / UTM Zone 45N (EPSG:32645)**. Save the project to your `projects/` directory.

#### Step 3: Permanent Reprojection
Reproject the vector river and district boundary layers to **EPSG:32645** using the **Reproject Layer** tool. Export the outputs as new layers within a GeoPackage file in your vector data folder. Do not run analysis on layers that remain in EPSG:4326.

#### Step 4: Styling
* **DEM Styling:** Style the elevation grid using a pseudocolor color ramp with classification breaks that highlight changes in terrain.

* **Rivers Styling:** Style river vector lines with a blue color ramp, scaling the line thickness based on stream hierarchy (e.g., wider lines for major mainstem channels, thinner lines for smaller headwater tributaries).

* **Boundaries Styling:** Style district boundaries with a semi-transparent fill and clean, dark borders.

#### Step 5: Layout Composition
1. Open the **Layout Manager** (**Project** > **New Print Layout**), naming it `Watershed_Layout`.

2. Configure page size to **A4 Landscape**.

3. Add the map canvas to the layout, adjusting scale and center alignment.

4. Add the following dynamic map elements:
   * **Title Block:** "Watershed Overview Map - WECS Study Area".
   * **Dynamic Scale Bar:** Configured in meters/kilometers with clean subdivision markers.
   * **North Arrow:** Positioned in a visible layout corner.
   * **Legend:** Clean layer labels (remove default underscores and raw filenames; e.g., use "Elevation (m)" instead of "basin_dem").
   * **Coordinates Grid:** Add a grid map frame overlay displaying coordinates in UTM coordinates with clean label formatting.

### Submission Instructions
Export your layout by clicking **Layout** > **Export as PDF**, saving the file to your `outputs/` folder. Submit a ZIP archive containing:

1. The exported map PDF (`Watershed_Overview_WECS.pdf`).

2. Your QGIS project file (`Day1_Assignment.qgz`).

3. The reprojected GeoPackage database file containing your styled vector layers.
