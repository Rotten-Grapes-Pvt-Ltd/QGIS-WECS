# Vector Geoprocessing Operations

Vector geoprocessing involves performing spatial algorithms on vector layers (points, lines, and polygons) to analyze proximity, intersection, containment, and geographic overlap. These operations are fundamental to catchment analysis, riparian zoning, and infrastructure planning.

In this section, we will utilize the **Natural Earth** vector datasets located in the following folders:

*   **Cultural 10m Data:** [docs/data/Natural_Earth_quick_start/10m_cultural/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_cultural/)
*   **Physical 10m Data:** [docs/data/Natural_Earth_quick_start/10m_physical/](file:///Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/10m_physical/)

---

## 1. Core Geoprocessing Algorithms

QGIS groups geoprocessing tools under the **Vector** > **Geoprocessing Tools** and **Vector** > **Analysis Tools / Geometry Tools** menus. Each tool performs a specific spatial boolean, proximity, or aggregation operation:

```text
    GEOPROCESSING OVERVIEW
    +-----------------+-----------------+-----------------+
    |     BUFFER      |      CLIP       |    DISSOLVE     |
    |    (Proximity)  | (Cookie Cutter) | (Merge Borders) |
    |      O -> ( )   |    [o] -> o     |   [ ]|[ ] -> [ ]|
    +-----------------+-----------------+-----------------+
    |    INTERSECT    |   DIFFERENCE    |      UNION      |
    |  (Overlap Only) | (Subtract Area) |  (All Combined) |
    |   (A)x(B) -> x  |   (A)-(B) -> ( )|   (A)+(B) -> AB |
    +-----------------+-----------------+-----------------+
    |    SYMMETRIC    |   MULTI-RING    |     SPATIAL     |
    |   DIFFERENCE    |     BUFFER      |      JOIN       |
    |  (A)XOR(B) -> X |  O -> ( ) ( )   |  Combine Tables |
    +-----------------+-----------------+-----------------+
```

*   **Buffer (Proximity):** Creates a polygon surrounding features at a specified distance. Used for setback zones, stream protection boundaries, or flood vulnerability envelopes.

*   **Multi-Ring Buffer:** Creates multiple concentric buffer rings around features at incremental distances (e.g., 100m, 500m, 1000m). Used for multi-tiered hazard zoning and proximity decay modeling.

*   **Clip (Spatial Filter):** Trims the features of an input layer to the exact boundary of a masking polygon layer (cookie-cutter). The attributes of the input layer are preserved, but the geometry is clipped.

*   **Dissolve (Aggregation):** Merges adjacent polygons that share an identical attribute value, removing the internal border lines between them. Crucial for aggregating sub-watersheds into major river basins.

*   **Intersect (Boolean AND):** Extracts only the overlapping portions of two input layers, joining attributes from both layers into the output features.

*   **Difference (Boolean NOT):** Subtracts the area of an overlay layer from the input layer, keeping only the portions of the input layer that do not overlap with the overlay layer.

*   **Symmetric Difference (Boolean XOR):** Extracts areas that belong to either the input layer or the overlay layer, but not both. Used to identify change/discrepancies between historical and current boundaries.

*   **Union (Boolean OR):** Combines the boundaries and attribute tables of both layers across their entire extent, generating new features for both overlapping and non-overlapping areas.

*   **Spatial Join (Join Attributes by Location):** Appends attributes from a join layer to a target layer based on their spatial relationship (intersects, contains, touches, within). This is a tabular operation powered by spatial overlays.

*   **Point-in-Polygon (Points in Polygon):** Counts the number of point features falling within each polygon boundary, and optionally calculates statistical summaries (e.g., sum, average) of point attributes for each polygon.

*   **Line-in-Polygon (Sum Line Lengths):** Calculates the total length of line features (e.g., rivers, roads) that fall within each polygon boundary, providing river density metrics per catchment.

*   **Centroids:** Generates a point feature at the geometric center of gravity of each input polygon.

*   **Point on Surface:** Generates a point feature guaranteed to fall inside the boundaries of the polygon (unlike centroids, which can fall outside for ring/doughnut or crescent-shaped polygons).

*   **Voronoi (Thiessen) Polygons:** Divides a space into regions based on proximity to a set of input points. Every location within a given polygon is closer to its associated input point (e.g., rain gauge) than to any other.

---

## 2. Step-by-Step Geoprocessing Exercises (Natural Earth)

The following exercises guide you through performing core and advanced vector geoprocessing operations using the localized Natural Earth datasets.

### Exercise 1: Buffer Major River Channels

Delineate a regional environmental protection buffer zone around the world's major river networks.

1. Load `ne_10m_rivers_lake_centerlines` (physical line layer) into QGIS.

2. Select only the major continental rivers: Go to **Select by Expression** (`Ctrl+F3`) and select features using: `"scalerank" <= 2`.

3. Open **Vector** > **Geoprocessing Tools** > **Buffer**.

4. Set the parameters:

   * **Input Layer:** `ne_10m_rivers_lake_centerlines` (check the box **Selected features only**).

   * **Distance:** `50` (set unit to **Kilometers**).

   * **Segments:** `5` (controls vertex smoothness).

   * Check the box **Dissolve result** to merge overlapping buffers from adjacent river sections.

   * **Buffered:** Save the dissolved output as a layer named `major_rivers_buffer_50km`.

5. Click **Run**. A continuous buffer corridor will appear surrounding the main river basins.

### Exercise 2: Clip the Road Network to Country Boundaries

Extract only the roads that lie within a specific country (e.g., Nepal) from the massive global roads shapefile.

1. Load `ne_10m_admin_0_countries` (cultural polygon layer) and `ne_10m_roads` (cultural line layer) into QGIS.

2. Filter the country layer for Nepal: Click `ne_10m_admin_0_countries`, open **Select by Expression**, and run: `"NAME" = 'Nepal'`.

3. Go to **Vector** > **Geoprocessing Tools** > **Clip**.

4. Set the parameters:

   * **Input Layer:** `ne_10m_roads` (the global network to cut).

   * **Overlay Layer:** `ne_10m_admin_0_countries` (check **Selected features only** to use the Nepal boundary).

   * **Clipped:** Save the output as a table in your GeoPackage named `nepal_clipped_roads`.

5. Click **Run**. The resulting layer will map only roads within Nepal.

### Exercise 3: Dissolve State Boundaries to Compile Country Borders

Rebuild nation borders by dissolving internal state/provincial lines.

1. Load `ne_10m_admin_1_states_provinces` (cultural polygon layer) into QGIS.

2. Go to **Vector** > **Geoprocessing Tools** > **Dissolve**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_admin_1_states_provinces`.

   * **Dissolve field(s):** Click `...` and check `admin` or `adm0_a3` (the attribute representing the parent country code).

   * **Dissolved:** Save the output as `dissolved_country_borders`.

4. Click **Run**. Observe that internal provincial boundaries are merged, leaving only compiled country outlines.

### Exercise 4: Intersect Urban Footprints with Country Boundaries

Identify where urban developments overlap with country lines and append national metadata to the urban polygons.

1. Load `ne_10m_urban_areas` (cultural polygon layer) and `ne_10m_admin_0_countries` (cultural polygon layer).

2. Go to **Vector** > **Geoprocessing Tools** > **Intersection**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_urban_areas`.

   * **Overlay Layer:** `ne_10m_admin_0_countries`.

   * **Intersection:** Save the output as `urban_intersection_countries`.

4. Click **Run**. Open the output's attribute table. Every urban polygon now carries both its local name and the country attributes (`NAME`, `CONTINENT`, `ISO_A3`) of the country it intersects.

### Exercise 5: Difference Lakes from Country Polygons

Calculate the net land area of countries by subtracting major waterbodies from country polygons.

1. Load `ne_10m_admin_0_countries` (cultural polygon layer) and `ne_10m_lakes` (physical polygon layer).

2. Go to **Vector** > **Geoprocessing Tools** > **Difference**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_admin_0_countries` (source land boundaries).

   * **Difference Layer:** `ne_10m_lakes` (waterbodies to subtract).

   * **Difference:** Save the output as `landmass_excluding_lakes`.

4. Click **Run**. The output layer will contain land outlines with holes where major lakes were located.

### Exercise 6: Multi-Ring Buffers for Flood Risk Zoning

Generate incremental vulnerability zoning rings surrounding major waterbodies.

1. Load `ne_10m_lakes` (physical polygon layer).

2. Open the **Processing Toolbox** (`Ctrl+Alt+T`) and search for **Multi-ring buffer (constant distance)**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_lakes`.

   * **Number of rings:** `3`.

   * **Distance between rings:** `20000` (set unit to **Meters** to create 20km, 40km, and 60km bands).

   * **Multi-ring buffer:** Save the output as `lakes_multiring_buffer_60km`.

4. Click **Run**. Change the symbology of the output layer to **Graduated** based on the `distance` attribute to visualize risk progression.

### Exercise 7: Spatial Join - Count Cities and Summarize Population by Country

Query points falling inside polygons and aggregate their attributes to the parent polygon features.

1. Load `ne_10m_populated_places` (cultural point layer) and `ne_10m_admin_0_countries` (cultural polygon layer).

2. In the **Processing Toolbox**, search for and open **Join Attributes by Location (Summary)**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_admin_0_countries` (polygons to receive attributes).

   * **Join Layer:** `ne_10m_populated_places` (points to be aggregated).

   * **Geometric predicate:** Select `contains`.

   * **Fields to summarize:** Click `...` and check `POP_MAX` (maximum population field).

   * **Summaries to calculate:** Check `count` and `sum`.

   * **Joined layer:** Save output as `countries_with_city_stats`.

4. Click **Run**. Open the attribute table of `countries_with_city_stats`. Find `POP_MAX_count` (total cities inside country) and `POP_MAX_sum` (combined maximum population of cities).

### Exercise 8: Line-in-Polygon - Calculate River Density by Country

Compute the cumulative length of lines falling inside corresponding polygon boundaries.

1. Load `ne_10m_rivers_lake_centerlines` (physical line layer) and `ne_10m_admin_0_countries` (cultural polygon layer).

2. Open the **Processing Toolbox** and search for **Sum line lengths**.

3. Set the parameters:

   * **Polygons:** `ne_10m_admin_0_countries`.

   * **Lines:** `ne_10m_rivers_lake_centerlines`.

   * **Line length field name:** Type `river_len`.

   * **Line count field name:** Type `river_cnt`.

   * **Line lengths:** Save the output as `countries_river_density`.

4. Click **Run**. Open the attribute table of the result. To calculate river density (km of river per sq. km of land area), open the **Field Calculator** and create a new field:

   * **Field Name:** `river_dens` (Decimal/Real type).

   * **Expression:** `("river_len" / 1000) / ($area / 1000000)`

### Exercise 9: Thiessen (Voronoi) Polygons for Areal Rainfall Estimation

Divide space based on proximity to coordinate points to model rain gauge influence zones.

1. Load `ne_10m_populated_places` into QGIS.

2. Select cities within Nepal: Select the layer, open **Select by Expression** (`Ctrl+F3`), and run: `"ADM0NAME" = 'Nepal'`.

3. Search for **Voronoi polygons** in the Processing Toolbox.

4. Set the parameters:

   * **Input Layer:** `ne_10m_populated_places` (check **Selected features only**).

   * **Buffer region (%):** `30` (extends polygon limits to cover the region boundary).

   * **Voronoi Polygons:** Save as `nepal_stations_thiessen`.

5. Click **Run**. Observe that the output polygons extend far beyond Nepal's borders.

6. Constrain the polygons to Nepal's boundary: Open **Vector** > **Geoprocessing Tools** > **Clip**.

   * **Input Layer:** `nepal_stations_thiessen`.

   * **Overlay Layer:** `ne_10m_admin_0_countries` (with Nepal selected, check **Selected features only**).

   * **Clipped:** Save as `nepal_thiessen_clipped`.

7. Click **Run**. You now have clean catchment-bounded Thiessen zones.

### Exercise 10: Symmetric Difference for Border Discrepancies

Identify non-overlapping regions between two similar boundary layers.

1. Load the `dissolved_country_borders` (output of Exercise 3) and `ne_10m_admin_0_countries`.

2. Open **Vector** > **Geoprocessing Tools** > **Symmetric Difference**.

3. Set the parameters:

   * **Input Layer:** `ne_10m_admin_0_countries`.

   * **Overlay Layer:** `dissolved_country_borders`.

   * **Symmetric Difference:** Save output as `boundary_discrepancies`.

4. Click **Run**. The output highlights any areas where state boundaries dissolved in Exercise 3 do not perfectly align with the national sovereign boundaries.

---

## 3. Hydrological Spotlight: Thiessen Polygons for Areal Precipitation

In hydrology, rain gauges provide point-based rainfall measurements. To calculate runoff in a watershed, we must convert these point observations into a single spatial average representing the **areal precipitation** ($P_{\text{avg}}$) over the entire catchment.

The **Thiessen Polygon Method** (also known as Voronoi tessellation) is one of the most widely used methods for this conversion.

```text
    THIESSEN VORONOI TESSELLATION
    +------------------------------+
    |   o  A1      |      o A2     |
    |              |               |
    |--------------+-----------    |
    |      |       o A3        \   |
    |  o   |                    \  |
    |  A4  |      o A5           \ |
    +------------------------------+
```

### Mathematical Formulation

The average precipitation is computed as a weighted average, where the weights are proportional to the polygon areas:

$$P_{\text{avg}} = \frac{\sum_{i=1}^{n} (P_i \times A_i)}{A_{\text{total}}} = \sum_{i=1}^{n} \left( P_i \times \frac{A_i}{A_{\text{total}}} \right)$$

Where:

*   $P_{\text{avg}}$ = Average catchment precipitation (mm).

*   $P_i$ = Precipitation recorded at gauge station $i$ (mm).

*   $A_i$ = Area of the Thiessen polygon corresponding to station $i$ contained within the catchment boundary ($\text{km}^2$).

*   $A_{\text{total}}$ = Total area of the catchment ($\text{km}^2$).

*   $W_i = A_i / A_{\text{total}}$ = Weight factor of station $i$.

### Sample Calculation Table

For a catchment of total area $1000\text{ km}^2$ with 4 stations:

| Station ID | Recorded Rainfall ($P_i$) | Polygon Area ($A_i$) | Weight ($W_i$) | Weighted Contribution ($P_i \times W_i$) |
| :--- | :--- | :--- | :--- | :--- |
| **ST_01** | 45 mm | $150\text{ km}^2$ | 0.15 | 6.75 mm |
| **ST_02** | 60 mm | $350\text{ km}^2$ | 0.35 | 21.00 mm |
| **ST_03** | 30 mm | $400\text{ km}^2$ | 0.40 | 12.00 mm |
| **ST_04** | 50 mm | $100\text{ km}^2$ | 0.10 | 5.00 mm |
| **Total** | **-** | **$1000\text{ km}^2$** | **1.00** | **$P_{\text{avg}} = 44.75\text{ mm}$** |

### Hydrological Limitations

*   **Orographic Precipitation:** The Thiessen method assumes linear variation in rainfall and does not account for elevation changes. If a gauge is located in a valley, it applies its rainfall value to nearby mountain peaks, potentially underestimating or overestimating high-altitude precipitation.

*   **Rigid Boundaries:** A small shift in station location changes the entire polygon layout, creating abrupt boundary jumps.

---

## 4. Topology Validation and Troubleshooting Geometry Errors

Geoprocessing tools require clean geometric inputs. When importing files or digitizing boundaries, datasets often contain hidden topological errors that cause geoprocessing algorithms to fail, hang, or output empty layers.

### Common Geometry Errors

*   **Self-Intersections (Bowties):** A polygon boundary crosses over itself, creating a loop. This is the most common cause of "Geoprocessing failed: Geometry is invalid" errors.

*   **Duplicate Nodes:** Two coordinate vertices placed at the exact same location in a sequence.

*   **Sliver Polygons:** Tiny, narrow gap spaces created between adjacent boundaries during manual editing.

*   **Holes and Overlaps:** Unintentional gap areas where adjacent polygons should snap together.

### How to Check and Repair Geometries in QGIS

1. **Verify Validity:** Search for **Check Validity** in the Processing Toolbox. Run it on your layer. It will output three temporary layers: *Valid Output*, *Invalid Output*, and *Error Locations*.

2. **Automated Repair:** Search for the **Fix Geometries** tool in the Processing Toolbox. This native QGIS tool runs automated topological repair routines (resolving self-intersections, repairing bowties, and closing unclosed rings).

3. **Snap Geometries:** Set up snapping rules in **Project** > **Snapping Options...** to ensure vertices align automatically during editing, preventing sliver creation.

### Enabling the Topology Checker Plugin

For complex workflows, QGIS includes a dedicated **Topology Checker** plugin to check relationship rules.

1. Go to **Plugins** > **Manage and Install Plugins...**.

2. Search for **Topology Checker** and check its box to activate it.

3. Open the plugin panel via **Vector** > **Topology Checker**.

4. Click **Configure** (wrench icon) to define rules:

   * For Admin Polygons: `must not overlap` and `must not have gaps`.

   * For Streams: `must not have dangles` (to find disconnected segments).

5. Click **Validate All** to find and inspect errors.

### Multipart vs. Singleparts Conversions

Dissolve and union operations often generate **Multipart geometries** (where a single database row contains multiple distinct physical polygons, such as islands belonging to a single country).

*   **Multipart to Singleparts:** Converts multi-polygons into individual rows. If a country has 5 islands, it splits them into 5 distinct database rows. Run this before calculating geometric centroids to ensure the points lie within the land area instead of in the ocean between islands.

*   **Singleparts to Multipart:** Groups features sharing a common ID back into a single database row.

---

## 5. Interactive Classroom Discussion Scenarios

### Scenario A: Catchment Buffer & Environmental Suitability Analysis

A hydrologist needs to identify potential zones for a new reservoir. The reservoir must be located within a specific district, must be within 2 km of a major river, but must be at least 500 meters away from all major roads and protected forest reserves. Outline the exact sequence of geoprocessing tools (Clip, Buffer, Difference, Intersect, Dissolve) required to isolate these suitable areas.

??? check "Answer Key - Suitability Sequence"

    1. **Dissolve** the protected forest reserves layer (if fragmented) to compile a single boundary.
    
    2. **Buffer** the roads layer by 500 meters (`roads_buffer_500m`).
    
    3. **Buffer** the rivers layer by 2 kilometers (`rivers_buffer_2km`).
    
    4. Combine the roads buffer and the dissolved forest reserves using **Union** or **Collect Geometries**, then **Dissolve** them to create a unified `exclusion_zone`.
    
    5. **Clip** the 2km river buffer by the district boundary polygon layer (`rivers_buffer_district`).
    
    6. Run **Difference** where the input layer is `rivers_buffer_district` and the difference layer is the `exclusion_zone`. The remaining polygons constitute the suitable zoning mask.

### Scenario B: Choice of Spatial Join - Stream Gauge Station Analysis

You have a layer of 50 watershed polygons and a layer of 200 daily river flow monitoring stations (points). You want to:

1. Assign to each monitoring station the name of the watershed it lies within.

2. Calculate the total number of monitoring stations in each watershed and their average daily flow value.

Which tools and configurations in the QGIS processing suite should you select for each task?

??? check "Answer Key - Spatial Join Selection"

    1. **For assigning watershed names to stations:** Use **Join attributes by location**.
       * **Base Layer:** Monitoring stations (points).
       * **Join Layer:** Watersheds (polygons).
       * **Geometric predicate:** Select `within` or `intersects`.
       * This outputs a point layer where each station's record contains the attributes of its parent watershed.
    
    2. **For calculating station counts and average flow per watershed:** Use **Join attributes by location (summary)**.
       * **Base Layer:** Watersheds (polygons).
       * **Join Layer:** Monitoring stations (points).
       * **Geometric predicate:** Select `contains` or `intersects`.
       * **Fields to summarize:** Select `daily_flow`.
       * **Summaries to calculate:** Check `count` and `mean`.
       * This outputs a polygon layer where each watershed contains new columns: `daily_flow_count` (number of stations) and `daily_flow_mean` (average daily flow).
