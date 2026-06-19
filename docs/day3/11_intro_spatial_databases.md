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

### Spinning Up PostGIS using Docker Compose

To quickly run a local PostGIS instance for development or training, you can spin it up inside a Docker container using a `docker-compose.yml` file.

Create a file named `docker-compose.yml` in your project folder with the following configuration:

```yaml
version: '3.8'

services:
  db:
    image: postgis/postgis:15-3.3
    container_name: postgis_db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: hydrology_db
      POSTGRES_USER: wecs_user
      POSTGRES_PASSWORD: securepassword123
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  pgdata:
```

#### How to Start and Verify:

1. Open a terminal in the folder containing `docker-compose.yml`.

2. Run the command to spin up the container in detached (background) mode:
   `docker compose up -d`

3. Verify that the container is running successfully:
   `docker ps`

---

## 3. Spatial SQL Syntax and Operations

PostGIS exposes spatial operators as standard SQL functions. By importing your Natural Earth layers (e.g., `countries`, `rivers`, and `cities` tables) into PostGIS, you can execute spatial queries directly inside the database engine:

### Area Calculations (`ST_Area`)

Calculates the polygon area. If using GEOMETRY, it outputs the value in projection units; if using GEOGRAPHY, it outputs the value in square meters:

```sql
-- Calculate area of South Asian countries in square kilometers
SELECT 
    name, 
    ST_Area(geom) / 1000000.0 AS area_sqkm 
FROM countries
WHERE subregion = 'Southern Asia';
```

### Proximity Analysis (`ST_Distance`)

Calculates the shortest distance separating two features. By casting geometry (`geom`) to the geography type, we get accurate geodesic distances in meters:

```sql
-- Find the distance in meters between Kathmandu and the Ganges river segment
SELECT 
    c.name AS city_name, 
    r.name AS river_name, 
    ST_Distance(c.geom::geography, r.geom::geography) AS distance_meters
FROM cities c, rivers r
WHERE c.name = 'Kathmandu' AND r.name = 'Ganges';
```

### Spatial Containment (`ST_Contains` / `ST_Within`)

Checks if a target geometry contains or is within another geometry (evaluates to true/false):

```sql
-- List all populated places (cities) located within the boundary of Nepal
SELECT 
    c.name AS city_name, 
    c.pop_max AS max_population
FROM cities c, countries co
WHERE co.name = 'Nepal' 
  AND ST_Contains(co.geom, c.geom)
ORDER BY c.pop_max DESC;
```

### Buffering & Intersections (`ST_Buffer`)

Generates a buffer around geometries at a specified metric distance:

```sql
-- Generate a 10km buffer corridor around major continental rivers
SELECT 
    name, 
    ST_Buffer(geom::geography, 10000)::geometry AS buffer_geom 
FROM rivers
WHERE scalerank <= 2;
```

---

## 4. Managing PostGIS Connections and Data in QGIS

Once the database is running inside Docker, QGIS provides integrated tools to browse, query, and edit PostGIS tables.

### Connecting QGIS to the Database:

1. In the QGIS **Browser Panel**, right-click **PostgreSQL** and select **New Connection...**.

2. Set the connection parameters:
   * **Name:** `WECS PostGIS Database`
   * **Host:** `localhost`
   * **Port:** `5432`
   * **Database:** `hydrology_db`
   * **Authentication:** Select **Basic** tab and enter Username `wecs_user` and Password `securepassword123`.

3. Click **Test Connection** to verify database access. Once connected, click **OK**.

4. The connection is now listed under **PostgreSQL** in the Browser Panel.

### Importing Vector Layers to PostGIS:

To upload your Natural Earth vector layers into the database for SQL analysis:

1. Go to **Database** > **DB Manager...** in the menu bar.

2. Expand **PostgreSQL** in the left panel and double-click the `WECS PostGIS Database` connection.

3. Click the **Import Layer/File** (down arrow) icon in the DB Manager toolbar.

4. Set the parameters:
   * **Input Layer:** Select the loaded vector layer you wish to upload (e.g., `ne_10m_rivers_lake_centerlines`).
   * **Schema:** Select `public`.
   * **Table:** Enter a clean, lowercase name (e.g., `rivers`).
   * **Options:** Check the boxes to **Create spatial index** (for GIST indexing), **Convert field names to lowercase**, and **Replace destination table if exists** (if re-importing).

5. Click **OK** to run the upload. Verify that the table appears under your database schema in the DB Manager and Browser lists.

### Running SQL Queries via DB Manager:

1. Expand the PostgreSQL listing in the DB Manager and select your database connection.

2. Click the **SQL Window** icon (a tablet with an SQL brush, or press `F2`).

3. Write your spatial SQL query (e.g., combining buffers and attribute selections):
   ```sql
   -- Find the total length of major rivers in kilometers grouped by class
   SELECT 
       featurecla, 
       COUNT(*) as segment_count,
       SUM(ST_Length(geom) / 1000.0) AS total_length_km
   FROM rivers
   GROUP BY featurecla;
   ```

4. Click **Execute** (`F5`) to view the results as a text table.

5. Check the box to **Load as new layer**. Select the geometry column (usually `geom`) and click **Load**. QGIS will immediately render the SQL results as a live vector layer on the Map Canvas.
