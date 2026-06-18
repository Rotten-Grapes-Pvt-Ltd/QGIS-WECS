# Spatial Databases and PostGIS

Flat files (like Shapefiles or GeoPackages) are inefficient for multi-user, enterprise-level geospatial workloads. Spatial databases resolve this issue by integrating GIS coordinate data and operations directly into database management systems (DBMS). This section covers PostgreSQL/PostGIS, spatial SQL querying, and database configuration inside QGIS.

---

## 1. RDBMS vs. Flat File Systems

Enterprise hydrology databases require support for concurrent editing, data integrity rules, and rapid query speeds across millions of coordinates.

| Parameter | Flat Files (Shapefiles / GPKGs) | Relational Databases (PostgreSQL / PostGIS) |
| :--- | :--- | :--- |
| **Multi-User Editing** | Single-user write access. File locks prevent simultaneous editing by a team. | Concurrent multi-user access with row-level locks, permitting team editing. |
| **Data Security** | Security is handled at the file level (anyone with the file can edit all values). | Granular permissions (read/write access rules per database table or user role). |
| **Relational Links** | Virtual joins only, slow to calculate on large datasets. | Enforces relational integrity rules (Primary and Foreign Keys) directly in the database. |

---

## 2. PostgreSQL and the PostGIS Extension

**PostgreSQL** is an enterprise-grade object-relational database. **PostGIS** is a spatial extension that adds support for geographic objects, spatial index grids (R-Tree / GIST), and spatial SQL functions to PostgreSQL.

### Spatial Data Types: Geometry vs. Geography

PostGIS distinguishes between two coordinate representations:

* **GEOMETRY:** Projects features onto a flat, Cartesian coordinate system (X, Y). Calculations (distance, area) are performed in the metric units of the underlying map projection (e.g., UTM meters). Best for regional analyses.

* **GEOGRAPHY:** Represents coordinates on a curved spheroidal model of the Earth (latitude, longitude). Calculations are calculated in geodesic meters and square meters, resolving distortions across continental bounds, but require significant computational overhead.

### Spatial Indexing (GIST)

PostgreSQL utilizes B-Tree indexes for fast text/numeric searches. PostGIS introduces **GIST (Generalized Search Tree)** spatial indexing. GIST indexes group feature coordinates into hierarchically nested bounding boxes. This allows the database to instantly locate features inside query zones without scanning every geometry row.

---

## 3. Spatial SQL Syntax and Operations

PostGIS exposes spatial operators as standard SQL functions. By writing SQL queries, you can run geoprocessing algorithms inside the database engine without rendering them on screen:

### Area Calculations (`ST_Area`)

Calculates the polygon area. If using GEOMETRY, it outputs value in projection units; if using GEOGRAPHY, it outputs value in square meters:

```sql
-- Calculate area in square kilometers for all catchments
SELECT 
    basin_name, 
    ST_Area(geom) / 1000000.0 AS area_sqkm 
FROM catchment_boundaries;
```

### Proximity Analysis (`ST_Distance`)

Calculates the shortest distance separating two features:

```sql
-- Find the distance in meters between gauges and planned reservoirs
SELECT 
    g.station_name, 
    r.project_name, 
    ST_Distance(g.geom, r.geom) AS distance_meters
FROM rain_gauges g, planned_reservoirs r;
```

### Spatial Containment (`ST_Contains` / `ST_Within`)

Checks if a target geometry contains or is within another geometry (evaluates to true/false):

```sql
-- List all rain gauges located inside the Koshi Basin polygon
SELECT 
    g.station_name 
FROM rain_gauges g, river_basins b
WHERE b.basin_name = 'Koshi' 
  AND ST_Contains(b.geom, g.geom);
```

### Buffering & Intersections (`ST_Buffer` & `ST_Intersection`)

Creates buffers and extracts intersecting overlapping geometries:

```sql
-- Generate a 100m vector riparian corridor polygon around streams
SELECT 
    river_name, 
    ST_Buffer(geom, 100) AS corridor_geom 
FROM river_network;
```

---

## 4. Connecting QGIS to PostGIS and DB Manager

QGIS provides integrated tools to browse, query, and edit PostGIS databases:

### Establishing a Connection:

1. In the **Browser Panel**, right-click **PostgreSQL** and select **New Connection...**.

2. Enter the connection parameters (Host, Port, Database, Username, and Password).

3. Click **Test Connection** to verify database access.

4. Once connected, expand the listing to drag and drop database tables directly onto the Map Canvas.

### Running SQL Queries via DB Manager:

1. Click **Database** > **DB Manager** > **DB Manager**.

2. Expand the PostgreSQL listing and select your connection schema.

3. Click the **SQL Window** icon (a tablet with an SQL brush).

4. Write your spatial SQL query (e.g., combining buffers and attribute selections).

5. Check the box to **Load as new layer**. Select the geometry column (usually `geom`) and click **Execute**. QGIS will immediately render the SQL results as a live vector layer on the Map Canvas.
