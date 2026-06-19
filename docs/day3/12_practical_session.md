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

---

## 7. Practical Session Exercises (Basic to Advanced)

The following 10 exercises test your geoprocessing, attribute querying, data conversion, and neighborhood analysis skills. They utilize various scales (110m, 50m, 10m) of cultural, physical, vector, and raster datasets available in your project directory.

### Basic Level

#### Exercise 1: Multi-Scale Layer Properties and Feature Counts
Analyze the effect of cartographic scale and generalization on vector geometries.

1. Load `ne_110m_admin_0_countries.shp` (coarse scale) and `ne_50m_admin_0_countries.shp` (medium scale) from [110m_cultural](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/110m_cultural/) and [50m_cultural](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/50m_cultural/) folders.

2. Open the **Layer Properties** > **Information** for both layers and identify the difference in coordinates precision and file sizes.

3. Open their Attribute Tables. Report the exact feature count (number of countries) contained in the 110m layer vs. the 50m layer.

#### Exercise 2: Projected Metric Lake Area Calculation
Calculate lake areas in projected metric units rather than spherical degrees.

1. Load the physical polygon layer `ne_50m_lakes.shp` from the [50m_physical](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/50m_physical/) folder.

2. Use the **Reproject Layer** tool to project the lakes into a metric CRS: **WGS 84 / UTM Zone 45N (EPSG:32645)**. Save the output as `lakes_utm.gpkg`.

3. Open the Field Calculator on `lakes_utm.gpkg` and create a new decimal field named `lake_area_sqkm`. Use the formula:
   `$area / 1000000`

4. Sort the table by `lake_area_sqkm` in descending order to identify the largest lake feature in the region.

#### Exercise 3: Landlocked African Country Attribute Selection
Perform attribute-based queries to filter specific spatial subsets.

1. Load `ne_110m_admin_0_countries.shp`.

2. Open **Select by Expression** (`Ctrl+F3`) and run the following query to select landlocked African countries with population over 10 million:
   ```sql
   "continent" = 'Africa' AND "pop_est" > 10000000 AND "featurecla" = 'Admin-0 country'
   ```
   *(Hint: You can check the attribute table structure to identify which columns contain landlocked flags or filter boundaries).*

3. Save the selected features to a new layer named `large_landlocked_africa.shp`.

#### Exercise 4: Continental Boundary Dissolve
Aggregate national political boundaries into global continental shapes.

1. Load `ne_50m_admin_0_countries.shp` into your canvas.

2. Go to **Vector** > **Geoprocessing Tools** > **Dissolve**.

3. Set the parameters:
   * **Input Layer:** `ne_50m_admin_0_countries.shp`.
   * **Dissolve field(s):** Check the box for `continent`.
   * **Dissolved:** Save to database as `continents_dissolved`.

4. Click **Run** and verify that internal national border lines are merged, leaving only continental outlines.

---

### Intermediate Level

#### Exercise 5: Riparian Buffer Corridors of Major Rivers
Create metric proximity buffers around primary stream centerlines.

1. Load `ne_50m_rivers_lake_centerlines_scale_rank.shp` from [50m_physical](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/50m_physical/).

2. Select the major continental rivers where scale rank is highest: `"scalerank" <= 2`.

3. Use **Reproject Layer** to reproject the selected river segments to **WGS 84 / UTM Zone 45N (EPSG:32645)** and save as `major_rivers_projected.gpkg`.

4. Run the **Buffer** tool on `major_rivers_projected.gpkg` with a distance of `20` **Kilometers**, checking the box for **Dissolve result**. Save the output as `river_buffers_20km`.

#### Exercise 6: Map Algebra Masking on Shaded Relief
Create a binary raster grid classifying shaded relief brightness values.

1. Load the shaded relief raster `NE1_50M_SR_W.tif` from the [50m_raster](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/50m_raster/NE1_50M_SR_W/) folder.

2. Open **Raster** > **Raster Calculator**.

3. Write an expression to isolate dark valley/shadow pixels (brightness value less than 100):
   ```sql
   "NE1_50M_SR_W@1" < 100
   ```

4. Save the output as `dark_relief_mask.tif`. Verify that the output pixels contain only $1\text{s}$ (shadows) and $0\text{s}$ (bright surfaces).

#### Exercise 7: Spatial Join of Cities and Countries
Transfer political attributes to point-based populated places by spatial location.

1. Load `ne_50m_populated_places.shp` (points) and `ne_50m_admin_0_countries.shp` (polygons).

2. Go to **Vector** > **Data Management Tools** > **Join Attributes by Location**.

3. Set the parameters:
   * **Input Layer:** `ne_50m_populated_places` (features to be joined to).
   * **Join Layer:** `ne_50m_admin_0_countries`.
   * **Geometric predicate:** Select **within**.
   * **Fields to add:** Click `...` and check `name` and `iso_a3`.
   * **Join type:** **Take attributes of the first matching feature**.
   * **Joined layer:** Save as `cities_joined_countries`.

4. Open the attribute table of `cities_joined_countries` and verify that the country details are appended to each city point.

---

### Advanced Level

#### Exercise 8: Zonal Statistics of Shaded Relief across South Asia
Calculate topographical texture summary statistics for South Asian countries.

1. Load the shaded relief raster `NE1_50M_SR_W.tif` and the countries vector `ne_50m_admin_0_countries.shp`.

2. Filter the countries layer to select South Asian countries: Open **Select by Expression** on the countries layer and run:
   ```sql
   "subregion" = 'Southern Asia'
   ```

3. Open **Zonal Statistics** in the Processing Toolbox.

4. Set the parameters:
   * **Raster Layer:** `NE1_50M_SR_W.tif`.
   * **Vector Layer containing zones:** `ne_50m_admin_0_countries` (check **Selected features only**).
   * **Output Column Prefix:** Enter `sr_`.
   * **Statistics to calculate:** Select **Mean**, **Min**, and **Max**.
   * **Zonal Statistics:** Save as `south_asia_relief_stats`.

5. Run the tool. Inspect the output table and identify which South Asian country contains the lowest minimum shaded brightness (indicating the deepest valleys/shadows).

#### Exercise 9: Vectorizing DEM Peaks and Clipping to Nepal
Extract high-altitude mountain peaks as vector polygons and clip them to national bounds.

1. Load the continuous DEM grid `output_hh.tif` from the [DEM](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/) folder and the high-resolution countries layer `ne_10m_admin_0_countries.shp` from [10m_cultural](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_cultural/).

2. Open **Raster Calculator** and generate a binary mask of peaks higher than $5500\text{ meters}$:
   ```sql
   "output_hh@1" > 5500
   ```
   Save the output as `peaks_mask_5500m.tif`.

3. Open **Raster** > **Conversion** > **Polygonize (Raster to Vector)**. Run it on `peaks_mask_5500m.tif` with a field name `is_peak` to create a polygon layer named `peaks_vector`.

4. Filter `peaks_vector` to keep only peak polygons: Select features where `"is_peak" = 1` and save them to a new layer `high_altitude_peaks_only`.

5. Filter `ne_10m_admin_0_countries.shp` for Nepal (`"NAME" = 'Nepal'`).

6. Run **Vector** > **Geoprocessing Tools** > **Clip** to clip the `high_altitude_peaks_only` polygons using the Nepal boundary polygon. Save the final output as `nepal_peaks_clipped`.

#### Exercise 10: Multi-Criteria Spatial Query and Proximity Matrix
Identify major cities located in close proximity to main continental river channels.

1. Load `ne_10m_populated_places.shp` (points) and `ne_10m_rivers_lake_centerlines.shp` (lines).

2. Select major cities with population greater than 1 million: Open **Select by Expression** on the populated places layer and run:
   ```sql
   "pop_max" > 1000000
   ```
   Save the selected features to a new layer named `major_cities`.

3. Select major rivers where scale rank is high: Open **Select by Expression** on the rivers layer and run:
   ```sql
   "scalerank" <= 3
   ```
   Save the selected segments to a new layer named `primary_rivers`.

4. Search for the **Distance to nearest hub (points)** tool in the Processing Toolbox.

5. Set the parameters:
   * **Source points layer:** `major_cities`.
   * **Destination hubs layer:** `primary_rivers`.
   * **Hub layer name attribute:** Select `name` (the river name field).
   * **Measurement unit:** Select **Kilometers**.
   * **Hub distance:** Save the output points as `cities_to_rivers_proximity`.

6. Run the tool. Open the attribute table of `cities_to_rivers_proximity`. Use **Select by Expression** to identify which major cities lie within $15\text{ km}$ of these major rivers:
   ```sql
   "HubDist" <= 15
   ```
