# Spatial Queries and Neighborhood Analysis

Unlike attribute selections, spatial queries filter, select, and aggregate datasets based on their geometric relationships—such as proximity, containment, intersection, and overlap. This section details spatial predicates, spatial joins, zonal statistics calculations, and proximity analyses.

We will use the localized **Natural Earth** vector layers and the digital elevation model grid **`output_hh.tif`**:

*   **Cultural 10m Data:** [docs/data/Natural_Earth_quick_start/10m_cultural/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_cultural/)

*   **Physical 10m Data:** [docs/data/Natural_Earth_quick_start/10m_physical/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_physical/)

*   **DEM Dataset:** [docs/data/Natural_Earth_quick_start/DEM/output_hh.tif](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/output_hh.tif)

---

## 1. Spatial Relationships (Spatial Predicates) & DE-9IM Theory

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
    |     CROSSES     |     EQUALS      |                 |
    |  (Line splits)  |  (Identical)    |                 |
    |     --/--       |     [ ]=[ ]     |                 |
    +-----------------+-----------------+-----------------+
```

### OGC Spatial Predicates

*   **Intersects:** Evaluates as `True` if two geometries share any coordinate point in common (including overlapping areas, crossing lines, or shared edge boundaries). This is the default spatial query.

*   **Contains:** Feature A *contains* Feature B if Feature B lies entirely within the boundary of Feature A, and their interiors intersect. For example, a catchment polygon *contains* rain gauge points.

*   **Within:** The exact inverse of contains. Feature A is *within* Feature B if Feature A lies entirely inside the boundary of Feature B.

*   **Disjoint:** Evaluates as `True` if two features share absolutely no coordinate space. They are completely separated.

*   **Touches:** Features touch if they share a border or vertex point, but their internal areas do not overlap. For example, two adjacent sub-catchment polygons touch along their shared watershed divide.

*   **Overlaps:** Features overlap if they share area space of the same dimension (e.g. two polygons) but neither contains the other.

*   **Crosses:** Feature A crosses Feature B if they share some but not all interior points, and the dimension of their intersection is less than the maximum dimension of either (e.g., a line crossing a polygon boundary).

*   **Equals:** Spatially identical; the coordinates of A and B are identical.

### Qualitative DE-9IM Theory

The mathematical engine under these predicates is the **Dimensionally Extended 9-Intersection Model (DE-9IM)**. It compares the **Interior (I)**, **Boundary (B)**, and **Exterior (E)** of geometry $A$ with those of geometry $B$ in a $3 \times 3$ matrix:

$$\text{DE-9IM}(A, B) = \begin{pmatrix} 
\dim(I(A) \cap I(B)) & \dim(I(A) \cap B(B)) & \dim(I(A) \cap E(B)) \\
\dim(B(A) \cap I(B)) & \dim(B(A) \cap B(B)) & \dim(B(A) \cap E(B)) \\
\dim(E(A) \cap I(B)) & \dim(E(A) \cap B(B)) & \dim(E(A) \cap E(B))
\end{pmatrix}$$

The values represent the dimension of the resulting intersection:

*   **`2`**: Area/Polygon

*   **`1`**: Line

*   **`0`**: Point

*   **`-1` (or `F`)**: Empty Set (No intersection)

*   **`T`**: Intersects (dimension $\ge 0$)

*   **`*`**: Any value allowed

For example, for two polygons to **Touch**, their interiors must not overlap ($\dim(I(A) \cap I(B)) = -1$), but their boundaries must intersect ($\dim(B(A) \cap B(B)) \ge 0$).

---

## 2. Step-by-Step Spatial Query Exercises (Natural Earth)

### Exercise 1: Select Cities near Major Rivers (Proximity Selection)

Select all major global cities (`ne_10m_populated_places`) located near major river buffer corridors.

1. Load `ne_10m_populated_places` (point layer) and the `major_rivers_buffer_50km` polygon layer (created in the previous section).

2. Go to **Vector** > **Research Tools** > **Select by Location**.

3. Set the parameters:
   
   * **Select features from:** `ne_10m_populated_places`.
   
   * **Where the features (geometric predicate):** Check **intersect**.
   
   * **By comparing to the features of:** `major_rivers_buffer_50km`.
   
   * **Modify current selection by:** Select **creating new selection**.

4. Click **Run**. The cities situated within $50\text{ km}$ of major river basins will be highlighted on the map canvas.

### Exercise 2: Compound Spatial-Attribute Selection

Select only the high-population cities (population $\ge 100,000$) inside Nepal that are within the river buffer zones.

1. Open **Select by Location**.
   
   * **Select features from:** `ne_10m_populated_places`.
   
   * **Where the features:** Check **intersect**.
   
   * **By comparing to:** `major_rivers_buffer_50km`.
   
   * **Modify current selection by:** Select **creating new selection**. Click **Run**.

2. Now, filter this spatial selection using attributes: Go to **Vector** > **Selection** > **Select by Expression**.

3. Set the parameters:
   
   * **Expression:** `"ADM0NAME" = 'Nepal' AND "POP_MAX" >= 100000`
   
   * **Modify current selection by:** Select **filter current selection** (this acts as a logical AND, restricting the current spatial selection by the attribute logic).

4. Click **Select Features**. QGIS will isolate only the major cities in Nepal located within the river buffer corridors.

---

## 3. Spatial Attribute Joins

A **Spatial Join** links the attribute tables of two layers based on their geographic location rather than a matching text key column.

| Join Type | Calculation Logic | Hydrological Use Case |
| :--- | :--- | :--- |
| **One-to-One** | Attaches attributes of the first matching feature to target features. | Assigning catchment attributes to point rain gauges. |
| **One-to-Many** | Creates duplicate target features for every matching source feature. | Assigning county boundaries to river segments that cross multiple boundaries. |
| **Summarized** | Aggregates all intersecting features and appends statistical metrics (sum, mean, count). | Counting flow gauges per river basin and calculating mean flow rates. |

*   **Accessing:** Go to **Vector** > **Data Management Tools** > **Join Attributes by Location**.

### Exercise 3: One-to-One Spatial Join

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

### Exercise 4: Summarized Spatial Join (One-to-Many)

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

### Exercise 5: Spatial Join of River Segments to Catchment Areas

Tag river line segments with the unique ID and name of the watershed polygon they intersect.

1. Load `ne_10m_rivers_lake_centerlines` (line) and `ne_10m_admin_0_countries` (polygon).

2. Open **Join Attributes by Location**.

3. Set the parameters:
   
   * **Input Layer:** `ne_10m_rivers_lake_centerlines` (lines to receive catchment data).
   
   * **Join Layer:** `ne_10m_admin_0_countries` (source polygons).
   
   * **Geometric predicate:** Select **intersects**.
   
   * **Fields to add:** Select `ISO_A3` and `NAME`.
   
   * **Join type:** Select **Take attributes of the first matching feature only (one-to-one)**.
   
   * **Joined layer:** Save as `rivers_with_country_tags`.

4. Click **Run**. The output line layer now contains country attributes, showing which nation manages each river stretch.

---

## 4. Zonal Statistics (Raster-Vector Overlay)

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

*   **Continuous vs. Categorical Zonal Stats:**
    
    *   For *Continuous Rasters* (e.g., DEM elevation or precipitation), statistical metrics like **Mean**, **Min**, **Max**, and **Std Dev** are useful.
    
    *   For *Categorical Rasters* (e.g., land cover zones), metrics like **Majority** (most common category inside the zone), **Minority** (least common), and **Variety** (number of unique classes) must be used.

### Exercise 6: Calculate State-Level Elevation Stats

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

A conservation group requires elevation summary statistics (mean, minimum, and maximum elevation) within a custom digitized Area of Interest (AOI) representing a proposed wildlife corridor. Follow these steps to digitize a custom boundary and extract its topographic metrics:

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

### Exercise 7: Zonal Histograms for Categorical Ecological Zones

Extract the distribution of ecological zones (Lowland, Mid-hills, Highland) inside Nepal's provinces.

1. Load `elevation_ecological_zones.tif` (created in Section 6 of the previous module) and `ne_10m_admin_1_states_provinces` (filtered to Nepal).

2. In the Processing Toolbox, search for and open **Zonal histogram**.

3. Set the parameters:
   
   * **Raster Layer:** `elevation_ecological_zones.tif`.
   
   * **Vector Layer containing zones:** `ne_10m_admin_1_states_provinces` (ensure **Selected features only** is checked).
   
   * **Output column prefix:** Enter `lc_`.
   
   * **Output Layer:** Save as `nepal_provinces_ecological_distribution`.

4. Click **Run**. Open the attribute table of the result. Note columns like `lc_1` (pixel counts of Lowland), `lc_2` (Mid-hills), and `lc_3` (Highland) added for each province.

---

## 5. Proximity Analysis and Distance Queries

Neighborhood analysis evaluates the geographic distance separating features across different layers.

*   **Distance Matrix:** Calculates the exact metric distance from each feature in a point layer to the nearest features in another layer.
    
    *   *Accessing:* Go to **Vector** > **Analysis Tools** > **Distance Matrix**.

*   **Distance to Nearest Hub (Hub Distance):** Calculates the distance between features in an input layer and the closest feature in a "destination" layer, appending the distance metric directly to the input attribute table.

### Exercise 8: Distance to Nearest Major River

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

### Exercise 9: Point-to-Point Distance Matrix for Hydromet Stations

Analyze the spatial density and distance separation of meteorological gauge stations.

1. Filter the points of `ne_10m_populated_places` to Nepal (as dummy rain gauge locations): Go to **Select by Expression** on the layer, run `"ADM0NAME" = 'Nepal'`, right-click the layer, select **Export** > **Save Selected Features As...**, and save as `nepal_gauge_stations.gpkg`.

2. Go to **Vector** > **Analysis Tools** > **Distance Matrix**.

3. Set the parameters:
   
   * **Input point layer:** `nepal_gauge_stations.gpkg`.
   
   * **Input unique ID field:** `NAME` (or another unique identifier field).
   
   * **Target point layer:** `nepal_gauge_stations.gpkg`.
   
   * **Target unique ID field:** `NAME`.
   
   * **Output matrix type:** Select **Linear (N x k) distance matrix**.
   
   * **Use only the nearest target points (k):** Enter `3` (this calculates the distance only to the 3 nearest neighboring stations, rather than the entire NxN network matrix).
   
   * **Distance Matrix:** Save output table as `gauge_network_proximity.dbf`.

4. Click **Run**. Open the table. The output matches each gauge with its 3 closest stations and lists the precise distance (in decimal degrees or meters, depending on CRS projection).

### Euclidean Distance Rasters (GDAL Proximity)

Instead of vector distance matrices, proximity can be represented as a continuous surface. The **Proximity (Raster Distance)** tool creates a raster where the value of each pixel represents the distance from that pixel to the closest feature in a vector or raster source layer (e.g. calculating a continuous distance grid from a river network to map flood corridors).

---

## 6. Interactive Classroom Discussion Scenarios

### Scenario A: Boundary Predicate Choice for Flood Risk Mapping

A municipal GIS engineer is calculating building inundation risks. They have:
*   A polygon layer representing building footprints.
*   A polygon layer representing the dissolved 100-year floodplain boundary.
They want to identify:
1. Buildings that are completely submerged inside the floodplain.
2. Buildings that are partially wet or touched by the floodplain boundary.
Which geometric predicates (Intersects, Contains, Within, Touches, Overlaps) should they select for each task?

??? check "Answer Key - Predicate Selection"

    1. **For completely submerged buildings:** Select buildings **Within** the floodplain. (Every coordinate of the building footprint lies strictly inside the floodplain boundary).
    
    2. **For partially wet/touched buildings:**
       * To find buildings that are partially inside but cross the boundary: Select buildings that **Overlap** the floodplain.
       * To find buildings that are built exactly along the outer edge but are dry: Select buildings that **Touch** the floodplain.
       * To catch all affected buildings (fully submerged, crossing, or touching): Select buildings that **Intersect** the floodplain.

### Scenario B: Boundary Rain Gauges in Catchment Aggregation

A hydrologist runs a summarized spatial join to calculate the count of rain gauges inside Catchment A. However, two crucial gauges are located directly on the boundary line between Catchment A and Catchment B. 
1. If they use the **Within** predicate, what will happen to these boundary gauges?
2. If they use the **Intersects** predicate, what will happen?
Explain the behavior based on DE-9IM boundary intersection rules and suggest a solution.

??? check "Answer Key - Boundary Gauge Aggregation"

    1. **Using the 'Within' Predicate:** Under DE-9IM rules, a point located exactly on the boundary of a polygon is not considered "Within" the polygon's interior. As a result, these two boundary gauges will be ignored entirely and will not be counted in either catchment.
    
    2. **Using the 'Intersects' Predicate:** Because a boundary point shares coordinate space with both polygons, it intersects both. A simple spatial count will count these gauges twice (once for Catchment A and once for Catchment B), inflating the total station count.
    
    3. **The Solution:** Use **Snapping** or a spatial rule where gauges are assigned to catchments based on **nearest centroid** (calculating distance from gauge to catchment centroids) or apply the **Join attributes by location** tool with a small tolerance (buffer) to assign boundary points based on the majority area. Alternatively, manual allocation must be done based on local drainage direction rules.
