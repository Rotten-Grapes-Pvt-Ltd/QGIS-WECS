# Practical Session: Geoprocessing, Table Operations, and Zonal Statistics

This hands-on session walks you through coordinate transformations, vector geoprocessing (buffering, clipping), calculating catchment stats using zonal statistics, and using the Field Calculator to compute geometric values.

---

## 1. Setting Up the Workspace and Data Inputs

Before beginning the analysis, establish your data links and workspace inside QGIS:

1. Open QGIS and create a new project. Save it as `Day3_Vector_Analysis.qgz` in your project folder.

2. In the **Browser Panel**, add a directory shortcut to your training folder.

3. Load the following input datasets:

   * `river_network.gpkg` (Linear stream network of study basin).

   * `sub_catchments.gpkg` (Polygon boundaries of catchment sub-zones).

   * `elevation_dem.tif` (Continuous elevation raster grid, e.g., Copernicus DEM).

---

## 2. Reprojecting Layers to a Projected Coordinate System

Before running proximity buffers or calculating polygon areas, datasets must use projected metric coordinates (meters) rather than spherical degrees (degrees).

1. Go to **Processing** > **Toolbox** and search for the **Reproject Layer** tool.

2. Run the tool on `river_network.gpkg`:

   * **Input Layer:** `river_network.gpkg`.

   * **Target CRS:** Select **WGS 84 / UTM Zone 45N (EPSG:32645)** (the appropriate projected system for Nepal).

   * Save the reprojected layer to `data/processed/vector/rivers_projected.gpkg`.

3. Repeat the reprojection steps for the `sub_catchments.gpkg` boundaries and save the output as `data/processed/vector/catchments_projected.gpkg`.

4. Remove the original unprojected layers from the Layers Panel to prevent formatting mistakes.

---

## 3. Creating a Dissolved Riparian Buffer Corridor

Delineate a $500\text{ meter}$ environmental protection buffer corridor surrounding all major rivers:

1. Navigate to **Vector** > **Geoprocessing Tools** > **Buffer**.

2. **Input Layer:** `rivers_projected.gpkg`.

3. **Distance:** Enter `500` (make sure the unit dropdown is set to **meters**).

4. **Segments:** Enter `5` (determines roundness of buffer corners).

5. Check the box for **Dissolve result**. This merges overlapping buffers from adjacent river sections into a single continuous polygon.

6. **Buffered:** Click `...` > **Save to GeoPackage**. Naming:
   * File path: `data/processed/vector/catchment_analysis.gpkg`
   * Table name: `riparian_buffer_500m`

7. Click **Run**. Toggle the layer styling to a semi-transparent blue fill to verify the buffer corridor surrounds the rivers correctly.

---

## 4. Clipping Geoprocessing Outputs

Clip the river buffer corridors to the specific boundary of a target sub-catchment to limit our study scope:

1. Navigate to **Vector** > **Geoprocessing Tools** > **Clip**.

2. **Input Layer:** `riparian_buffer_500m` (the layer to be cut).

3. **Overlay Layer:** `catchments_projected.gpkg` (the polygon boundary acting as the cookie-cutter).

4. **Clipped:** Click `...` > **Save to GeoPackage** inside `catchment_analysis.gpkg`. Naming the table: `study_corridor_clipped`.

5. Click **Run**. A new layer will appear showing only the buffers that fall inside the catchment boundary.

---

## 5. Zonal Statistics Laboratory

Extract topographic elevations for each sub-catchment polygon from the digital elevation grid:

1. Search for **Zonal Statistics** in the Processing Toolbox.

2. Set the tool parameters:

   * **Raster Layer:** `elevation_dem.tif`.

   * **Vector Layer containing zones:** `catchments_projected.gpkg`.

   * **Output Column Prefix:** Enter `dem_` (this flags the generated columns).

3. Click **Statistics to calculate** (`...`) and check **Mean**, **Min**, and **Max**.

4. **Zonal Statistics (Output):** Set to save the updated table to a new table `catchments_elev_stats` inside `catchment_analysis.gpkg`.

5. Click **Run**.

6. Right-click the newly created `catchments_elev_stats` layer in the Layers panel and select **Open Attribute Table**. Scroll to the far right to verify that three new columns (`dem_mean`, `dem_min`, `dem_max`) contain the extracted elevations for each catchment.

---

## 6. Dynamic Area Calculations via the Field Calculator

Calculate the total area in square kilometers ($km^2$) for each catchment polygon using database equations:

1. Open the attribute table of the `catchments_elev_stats` layer.

2. Click the **Toggle Editing Mode** icon (the pencil icon, or press `Ctrl+E`).

3. Click the **Open Field Calculator** icon (the abacus icon, or press `Ctrl+I`).

4. Configure the Field Calculator parameters:

   * Check **Create a new field**.

   * **Output field name:** Enter `area_sqkm`.

   * **Output field type:** Select **Decimal number (real)**.

   * **Precision:** Set to `2` (places after the decimal).

5. In the expression editor panel, enter the geometry calculation formula:
   `$area / 1000000`

6. Click **OK**. QGIS will compute and add the area values to every row.

7. Click the **Save Edits** icon and press `Ctrl+E` to toggle editing mode off.
