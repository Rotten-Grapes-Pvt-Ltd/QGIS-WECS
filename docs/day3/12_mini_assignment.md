# Mini-Assignment: Building a Hydrological GIS Database

In this assignment, you will apply the skills learned during Day 3 to clean vector datasets, execute spatial joins, calculate catchment statistics, compute geometric attributes, and compile a formatted layout map. This assignment is designed to require approximately 1 hour to complete.

---

## 1. Objective

Build a structured, centralized hydrological database inside a single GeoPackage file. You will calculate sub-basin average elevations, stream network lengths, and riparian buffer corridors, and export a cartographically sound print layout map illustrating your analytical results.

---

## 2. Provided Datasets

The following raw layers are located in your project directory:

* `/data/vector/raw_districts.shp` (Administrative boundary polygons, EPSG:4326).

* `/data/vector/river_branches.gpkg` (Polyline stream centerlines, EPSG:4326).

* `/data/raster/terrain_elevation.tif` (Digital elevation model raster grid, EPSG:4326).

---

## 3. Step-by-Step Requirements

### Step 1: Database Initialization and Reprojection

1. In the QGIS Browser panel, right-click a local folder and create a new OGC GeoPackage database file named `Basin_Hydrology_Database.gpkg`.

2. Import the raw `raw_districts.shp` and `river_branches.gpkg` layers.

3. Reproject both layers into the metric UTM Zone 45N projection (**EPSG:32645**) to ensure all calculations use meter units. Store the reprojected tables inside `Basin_Hydrology_Database.gpkg` as:

   * `district_boundaries`

   * `river_centerlines`

---

### Step 2: Proximity Buffer Analysis

Delineate a riparian conservation corridor around the river network:

1. Run the **Buffer** tool on `river_centerlines`.

2. Set the buffer distance to **$250\text{ meters}$**.

3. Check the box to **Dissolve result** to merge overlapping stream buffers.

4. Save the output table directly inside your GeoPackage database as `riparian_buffer_250m`.

---

### Step 3: Zonal Statistics and Attribute Calculations

Extract elevation statistics and calculate geographic metrics for the administrative districts:

1. Run the **Zonal Statistics** tool:

   * **Raster Layer:** `terrain_elevation.tif`.

   * **Vector Layer (Zones):** `district_boundaries`.

   * **Output Prefix:** `elev_`.

   * **Statistics:** Calculate `Mean` and `Max`.

2. Save the output table in your GeoPackage as `districts_elevation_summary`.

3. Open the attribute table of `districts_elevation_summary`. Turn on editing mode (`Ctrl+E`) and open the **Field Calculator** (`Ctrl+I`):

   * Create a new field named `area_sqkm` (Decimal number, Precision: 2). Formula: `$area / 1000000`

   * Save changes and toggle editing mode off.

---

### Step 4: Map Layout and Cartography

Create a publication-quality map of your study area:

1. Open a new **Print Layout** and set the page size to **A4 Landscape**.

2. Add a map frame rendering the following layers:

   * **Top Layer:** `riparian_buffer_250m` styled in semi-transparent light blue (Hex `#4a90e2`) with a dashed border.

   * **Middle Layer:** `river_centerlines` styled as thin blue lines.

   * **Bottom Layer:** `districts_elevation_summary` styled using **Graduated Symbology** based on the `elev_mean` field. Apply a *Viridis* or *YlOrRd* color ramp, classified into $5$ bins using **Jenks Natural Breaks**.

3. Add the following cartographic elements to the layout page:

   * **Title Block:** "Administrative District Elevation and Riparian Buffer Map".

   * **Scale Bar:** Dynamic metric scale bar (labeled in km).

   * **North Arrow:** Placed in an open quadrant.

   * **Legend:** Clear labels for each active layer. Clean up raw column name entries (e.g. rename `elev_mean` range bins to "Average Elevation (meters)").

   * **Map Grid (Graticule):** Add coordinate lines displayed in UTM meter values along the map borders.

---

## 4. Submission Instructions

Export your layout map as a PDF. Compile a ZIP archive named `Day3_Assignment_[YourName].zip` containing:

1. The exported PDF map (`District_Elevation_Buffer_Map.pdf`).

2. The completed GeoPackage database container (`Basin_Hydrology_Database.gpkg`) containing all four tables.

3. The QGIS project configuration file (`Day3_Assignment.qgz`) saved with relative data paths.
