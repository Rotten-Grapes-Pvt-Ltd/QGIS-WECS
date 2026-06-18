# Spatial Queries and Neighborhood Analysis

Unlike attribute selections, spatial queries filter, select, and aggregate datasets based on their geometric relationships—such as proximity, containment, intersection, and overlap. This section details spatial predicates, spatial joins, zonal statistics calculations, and proximity analyses.

We will use the localized **Natural Earth** vector layers and the digital elevation model grid **`output_hh.tif`**:

*   **Cultural 10m Data:** [docs/data/Natural_Earth_quick_start/10m_cultural/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_cultural/)
*   **Physical 10m Data:** [docs/data/Natural_Earth_quick_start/10m_physical/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_physical/)
*   **DEM Dataset:** [docs/data/Natural_Earth_quick_start/DEM/output_hh.tif](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/output_hh.tif)

---

## 1. Spatial Relationships (Spatial Predicates)

Spatial predicates are boolean queries evaluating the coordinate overlap between two features. QGIS utilizes standard OGC spatial predicates inside the **Select by Location** tool (**Vector** > **Research Tools** > **Select by Location**):

```text
    SPATIAL GEOMETRIC RELATIONS
    +-----------------+-----------------+-----------------+
    |   INTERSECTS    |    CONTAINS     |     WITHIN      |
    |   (Any touch)   |   (All inside)  |   (Is inside)   |
    |      O---O      |     [  o  ]     |     o [   ]     |
    +-----------------+-----------------+-----------------+
    |    DISJOINT     |     TOUCHES     |    OVERLAPS     |
    |  (No contact)   |  (Shared edge)  | (Partial cross) |
    |   O   O         |     [ ][ ]      |     [ o ]       |
    +-----------------+-----------------+-----------------+
```

* **Intersects:** Evaluates as `True` if two geometries share any coordinate point in common (including overlapping areas, crossing lines, or shared edge boundaries). This is the default spatial query.
* **Contains:** Feature A *contains* Feature B if Feature B lies entirely within the boundary of Feature A. For example, a country polygon *contains* city points.
* **Within:** The exact inverse of contains. Feature A is *within* Feature B if Feature A lies entirely inside the boundary of Feature B.
* **Disjoint:** Evaluates as `True` if two features share absolutely no coordinate space. They are completely separated.
* **Touches:** Features touch if they share a border or vertex point, but their internal areas do not overlap. For example, two adjacent sub-catchment polygons touch along their shared watershed divide.
* **Overlaps:** Features overlap if they share area space but neither contains the other.

### Exercise: Select Cities near Major Rivers (Proximity Selection)
Select all major global cities (`ne_10m_populated_places`) located near major river buffer corridors.

1. Load `ne_10m_populated_places` (point layer) and the `major_rivers_buffer_50km` polygon layer (created in the previous section).
2. Go to **Vector** > **Research Tools** > **Select by Location**.
3. Set the parameters:
   * **Select features from:** `ne_10m_populated_places`.
   * **Where the features (geometric predicate):** Check **intersect**.
   * **By comparing to the features of:** `major_rivers_buffer_50km`.
   * **Modify current selection by:** Select **creating new selection**.
4. Click **Run**. The cities situated within $50\text{ km}$ of major river basins will be highlighted on the map canvas.

---

## 2. Spatial Attribute Joins

A **Spatial Join** links the attribute tables of two layers based on their geographic location rather than a matching text key column.

* **Accessing:** Go to **Vector** > **Data Management Tools** > **Join Attributes by Location**.

### Exercise 1: One-to-One Spatial Join
Assign country name details to point cities based on the country boundaries they fall within.

1. Load `ne_10m_populated_places` (point) and `ne_10m_admin_0_countries` (polygon).
2. Open **Vector** > **Data Management Tools** > **Join Attributes by Location**.
3. Set the parameters:
   * **Input Layer (Target):** `ne_10m_populated_places`.
   * **Join Layer (Source):** `ne_10m_admin_0_countries`.
   * **Geometric predicate:** Select **within**.
   * **Fields to add:** Select `NAME` and `CONTINENT`.
   * **Join type:** Select **Take attributes of the first matching feature only (one-to-one)**.
   * **Joined layer:** Save as `cities_with_country_names`.
4. Click **Run**. Verification: Open the output's attribute table to confirm each city now maps its country name.

### Exercise 2: Summarized Spatial Join (One-to-Many)
Aggregate city points into country polygons to count how many major cities are in each country and find the maximum city population.

1. Open **Join Attributes by Location**.
2. Set the parameters:
   * **Input Layer:** `ne_10m_admin_0_countries`.
   * **Join Layer:** `ne_10m_populated_places`.
   * **Geometric predicate:** Select **contains**.
   * **Fields to add:** Select `pop_max` (the city population column).
   * **Join type:** Select **Take summary of matching features (one-to-many)**.
   * **Summaries to calculate:** Check **count** (gives city counts per country) and **max** (gives largest city population).
   * **Joined layer:** Save as `countries_city_summaries`.
3. Click **Run**. Look at the generated columns `pop_max_count` and `pop_max_max` in the output layer.

---

## 3. Zonal Statistics (Raster-Vector Overlay)

**Zonal Statistics** calculates statistical summary metrics (mean, maximum, minimum, standard deviation, sum) of a raster grid within the boundaries defined by a vector polygon layer.

```text
    +----------------------------------+
    |   Raster Grid (e.g. DEM)         |
    |   [2500][2600][2700][2800][2900] |
    +----------------------------------+
         |
         |  ZONAL OVERLAY (State / Province Polygon)
         v
    +----------------------------------+
    | Calculated Vector Output         |
    | dem_mean Elevation  = 2700.0 m   |
    | dem_min Elevation   = 2500.0 m   |
    | dem_max Elevation   = 2900.0 m   |
    +----------------------------------+
```

* **Accessing:** Open the Processing Toolbox and search for **Zonal Statistics**.

### Exercise: Calculate State-Level Elevation Stats
Extract mean, minimum, and maximum elevations for the provinces of Nepal using the `output_hh.tif` DEM grid and the state boundaries vector.

1. Load `ne_10m_admin_1_states_provinces` (polygon) and `output_hh.tif` (continuous DEM).
2. Filter the state boundaries to Nepal: Open **Select by Expression** on the states layer and run: `"admin" = 'Nepal'`.
3. Search for **Zonal Statistics** in the Processing Toolbox.
4. Set the parameters:
   * **Raster Layer:** `output_hh.tif` (the elevation grid).
   * **Vector Layer containing zones:** `ne_10m_admin_1_states_provinces` (check the box **Selected features only**).
   * **Output Column Prefix:** Enter `dem_`.
   * **Statistics to calculate:** Click `...` and check **Mean**, **Min**, and **Max**.
   * **Zonal Statistics:** Save the output table as `nepal_states_elevation_stats`.
5. Click **Run**.
6. Open the attribute table of the output layer. Scroll to the far right to see the columns `dem_mean`, `dem_min`, and `dem_max` showing elevation stats for each Nepal province.

### Problem Statement: Elevation Stats for a Digitized Custom Study Area (AOI)

**Scenario:** A conservation group requires elevation summary statistics (mean, minimum, and maximum elevation) within a custom digitized Area of Interest (AOI) representing a proposed wildlife corridor. Follow these steps to digitize a custom boundary and extract its topographic metrics:

1. **Create a Custom Polygon Layer:**
   * Go to **Layer** > **Create Layer** > **New Temporary Scratch Layer...**.
   * **Layer name:** Enter `study_area_aoi`.
   * **Geometry type:** Select **Polygon**.
   * **CRS:** Match the projected coordinate system of the DEM (`output_hh_utm_30m.tif`, EPSG:32645). Click **OK**.

2. **Digitize the Boundary:**
   * Click the `study_area_aoi` layer in the Layers panel, then click the **Toggle Editing** (pencil) icon.
   * Select the **Add Polygon Feature** (`Ctrl+.`) tool.
   * Click on the map canvas to draw a polygon outlining a custom corridor through the valleys and ridges of the DEM. Right-click to close and complete the polygon.
   * Click the **Save Layer Edits** icon and toggle editing mode off.

3. **Calculate Zonal Statistics:**
   * Open the **Zonal Statistics** tool from the Processing Toolbox.
   * Set the parameters:
     * **Raster Layer:** `output_hh.tif` (or `output_hh_utm_30m.tif`).
     * **Vector Layer containing zones:** `study_area_aoi` (your custom digitized polygon).
     * **Output Column Prefix:** Enter `aoi_`.
     * **Statistics to calculate:** Select **Mean**, **Min**, and **Max**.
   * Click **Run**.

4. **Review Results:**
   * Open the attribute table of `study_area_aoi`. The columns `aoi_mean`, `aoi_min`, and `aoi_max` contain the calculated metrics representing the topography of your custom digitized corridor.

---

## 4. Proximity Analysis and Distance Queries

Neighborhood analysis evaluates the geographic distance separating features across different layers.

* **Distance Matrix:** Calculates the exact metric distance from each feature in a point layer to the nearest features in another layer.
  * *Accessing:* Go to **Vector** > **Analysis Tools** > **Distance Matrix**.
* **Distance to Nearest Hub (Hub Distance):** Calculates the distance between features in an input layer and the closest feature in a "destination" layer, appending the distance metric directly to the input attribute table.

### Exercise: Distance to Nearest Major River
Calculate the distance from every global city (`ne_10m_populated_places`) to the nearest major river centerline.

1. Load `ne_10m_populated_places` (point) and `ne_10m_rivers_lake_centerlines` (line).
2. In the Processing Toolbox, search for the **Distance to nearest hub (points)** tool.
3. Set the parameters:
   * **Source points layer:** `ne_10m_populated_places`.
   * **Destination hubs layer:** `ne_10m_rivers_lake_centerlines`.
   * **Hub layer name attribute:** Select `name` (attaches the name of the nearest river to the city).
   * **Measurement unit:** Select **Kilometers** (or **Meters** if using a projected system).
   * **Hub distance:** Save the output point layer as `cities_river_proximity`.
4. Click **Run**. Open the attribute table of the output layer. You will find the fields `HubName` (river name) and `HubDist` (distance to river) appended to every city point.
