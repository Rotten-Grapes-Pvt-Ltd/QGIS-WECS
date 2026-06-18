# Spatial Queries and Neighborhood Analysis

Unlike attribute selections, spatial queries filter, select, and aggregate datasets based on their geometric relationships—such as proximity, containment, intersection, and overlap. This section details spatial predicates, spatial joins, zonal statistics calculations, and proximity analyses.

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

* **Contains:** Feature A *contains* Feature B if Feature B lies entirely within the boundary of Feature A. For example, a watershed polygon *contains* a rain gauge point.

* **Within:** The exact inverse of contains. Feature A is *within* Feature B if Feature A lies entirely inside the boundary of Feature B.

* **Disjoint:** Evaluates as `True` if two features share absolutely no coordinate space. They are completely separated.

* **Touches:** Features touch if they share a border or vertex point, but their internal areas do not overlap. For example, two adjacent sub-catchment polygons touch along their shared watershed divide.

* **Overlaps:** Features overlap if they share area space but neither contains the other.

---

## 2. Spatial Attribute Joins

A **Spatial Join** links the attribute tables of two layers based on their geographic location rather than a matching text key column.

* **Accessing:** Go to **Vector** > **Data Management Tools** > **Join Attributes by Location**.

### Query Styles

* **One-to-One Spatial Join:** Appends the attributes of the container polygon to the points inside. For example, joining a "Districts" polygon layer to a "Rain Gauges" point layer to assign a `District_Name` column to each gauge point.

* **Summarized Spatial Join (One-to-Many):** Aggregates values from multiple intersecting features. For example, joining "Landslide Points" to "Basin Polygons" to count how many landslides occurred in each basin. QGIS can compute statistical summaries like `count`, `sum`, `mean`, or `maximum` of the joined values.

---

## 3. Zonal Statistics (Raster-Vector Overlay)

**Zonal Statistics** calculates statistical summary metrics (mean, maximum, minimum, standard deviation, sum) of a raster grid within the boundaries defined by a vector polygon layer.

```text
    +------------------------------+
    |   Raster Grid (e.g. DEM)     |
    |   [250][260][270][280][290]  |
    +------------------------------+
         |
         |  ZONAL OVERLAY (Sub-catchment Polygon)
         v
    +------------------------------+
    | Calculated Vector Output     |
    | Mean Elevation  = 270.0 m    |
    | Min Elevation   = 250.0 m    |
    | Max Elevation   = 290.0 m    |
    +------------------------------+
```

* **Hydrological Application:** Overlaying a monthly precipitation raster grid on top of sub-catchment polygons to calculate the average rainfall volume ($mm$) entering each basin.

* **Accessing:** Search for **Zonal Statistics** in the Processing Toolbox.

### Setup Settings:

1. **Raster Layer:** The continuous grid containing the values you want to summarize (e.g., Copernicus DEM or CHIRPS precipitation).

2. **Vector Layer:** The polygon boundaries defining the summary zones (e.g., sub-basins).

3. **Output Column Prefix:** A short text prefix (e.g., `dem_`) to distinguish the new columns in the vector attribute table.

4. **Statistics to Calculate:** Check the boxes for the specific metrics you need (e.g., `Mean`, `Min`, `Max`).

---

## 4. Proximity Analysis and Distance Queries

Neighborhood analysis evaluates the geographic distance separating features across different layers.

* **Distance Matrix:** Calculates the exact metric distance from each feature in a point layer to the nearest features in another layer.
  * *Accessing:* Go to **Vector** > **Analysis Tools** > **Distance Matrix**.
  * *Hydrological Application:* Finding the nearest water quality monitoring station to a planned reservoir intake.

* **Hub Distance (Distance to Nearest Hub):** Calculates the distance between features in an input layer and the closest feature in a "destination" layer, appending the distance metric directly to the input attribute table.
