# Mini-Assignment: Delineating a Watershed Layout Map

In this assignment, you will apply the skills learned during Day 1 to set up a structured GIS workspace, import datasets, perform coordinate transformations, and compile a formatted map layout for submission.

---

## 1. Objective
Produce a georeferenced, styled watershed overview map for a selected catchment in Nepal, formatted on an $A4$ sheet, and export the output as a high-quality PDF.

---

## 2. Provided Datasets

* `/data/vector/nepal_districts.gpkg` (Administrative divisions of Nepal, EPSG:4326).

* `/data/vector/major_rivers.gpkg` (Stream centerlines, EPSG:4326).

* `/data/raster/basin_dem.tif` (ALOS PALSAR Digital Elevation Model, EPSG:4326).

---

## 3. Step-by-Step Requirements

### Step 1: Workspace Organization
Create a directory structure on your computer named `WECS_Assignment_Day1` containing subdirectories for `data`, `projects`, and `outputs`. Copy the provided datasets into their respective folders.

### Step 2: Project CRS Setup
Open QGIS, create a new project, and configure the project CRS to **WGS 84 / UTM Zone 45N (EPSG:32645)**. Save the project to your `projects/` directory.

### Step 3: Permanent Reprojection
Reproject the vector river and district boundary layers to **EPSG:32645** using the **Reproject Layer** tool. Export the outputs as new layers within a GeoPackage file in your vector data folder. Do not run analysis on layers that remain in EPSG:4326.

### Step 4: Styling

* **DEM Styling:** Style the elevation grid using a pseudocolor color ramp with classification breaks that highlight changes in terrain.

* **Rivers Styling:** Style river vector lines with a blue color ramp, scaling the line thickness based on stream hierarchy (e.g., wider lines for major mainstem channels, thinner lines for smaller headwater tributaries).

* **Boundaries Styling:** Style district boundaries with a semi-transparent fill and clean, dark borders.

### Step 5: Layout Composition

1. Open the **Layout Manager** (**Project** > **New Print Layout**), naming it `Watershed_Layout`.

2. Configure page size to **A4 Landscape**.

3. Add the map canvas to the layout, adjusting scale and center alignment.

4. Add the following dynamic map elements:

   * **Title Block:** "Watershed Overview Map - WECS Study Area".

   * **Dynamic Scale Bar:** Configured in meters/kilometers with clean subdivision markers.

   * **North Arrow:** Positioned in a visible layout corner.

   * **Legend:** Clean layer labels (remove default underscores and raw filenames; e.g., use "Elevation (m)" instead of "basin_dem").

   * **Coordinates Grid:** Add a grid map frame overlay displaying coordinates in UTM coordinates with clean label formatting.

---

## 4. Submission Instructions
Export your layout by clicking **Layout** > **Export as PDF**, saving the file to your `outputs/` folder. Submit a ZIP archive containing:

1. The exported map PDF (`Watershed_Overview_WECS.pdf`).

2. Your QGIS project file (`Day1_Assignment.qgz`).

3. The reprojected GeoPackage database file containing your styled vector layers.
