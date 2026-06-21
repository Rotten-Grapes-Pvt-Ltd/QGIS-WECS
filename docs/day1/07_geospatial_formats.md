# Geospatial Data Formats

Geospatial files must store coordinate geometries alongside descriptive attributes. Different formats have been optimized for different types of spatial analysis, data storage, and web sharing. This section explains the limitations of legacy shapefiles and the advantages of modern formats like GeoPackage, GeoJSON, and Cloud-Optimized GeoTIFF.


!!! tip  "Presentation Slides"
    You can download or view the lecture slides for this topic: [Modernizing_Geospatial_formats.pdf](presentations/07_Modernizing_Geospatial_formats.pdf)

---

## 1. The Legacy Shapefile and Its Limitations
Developed by ESRI in the 1990s, the **Shapefile (.shp)** was the industry standard for decades. However, it is a legacy format with severe structural limitations:

1. **Multi-File Requirement:** A shapefile is not a single file, but a collection of at least three mandatory files (`.shp` for geometries, `.dbf` for attributes, and `.shx` for the spatial index). If any of these files are missing or renamed, the dataset becomes corrupted.

2. **Column Name Limit:** Column headers in the attribute table are limited to **10 characters** (defined by the legacy dBase IV format).

3. **File Size Limit:** The total size of any individual component file cannot exceed **2 GB**.

4. **No Multi-Geometry Type:** A single shapefile can only store one geometry type (e.g., points or lines, but not both in the same file).

5. **No Date/Time and Null Support:** Poor support for null values and coordinate time stamps, which are essential for storing time-series sensor data.

---

## 2. The Modern GeoPackage (.gpkg)
The **GeoPackage** is the modern replacement for shapefiles. It is an open format standardized by the Open Geospatial Consortium (OGC).

* **Single-File Convenience:** An entire database is stored in a single SQLite file, making it easy to share.

* **Multi-Layer Support:** A single `.gpkg` file can store multiple vector layers, raster grids, non-spatial attribute tables, and cartographic styles.

* **No Table Limits:** No limit on column name lengths or table sizes (other than hardware limits).

* **Database Enabled:** Supports SQL triggers, views, and spatial indexing out-of-the-box, allowing quick queries on large vector layers.

---

## 3. GeoJSON (Geographic JSON)
GeoJSON is an open, text-based format designed for web mapping applications and API data exchange.

* **Format:** Based on JSON (JavaScript Object Notation). It represents geometries, coordinate lists, and attributes as plain text.

* **Default CRS:** Always uses **WGS 84 (EPSG:4326)** coordinates.

* **Web Native:** Can be parsed natively by web browsers and mapping libraries like Leaflet, OpenLayers, and Mapbox GL.

* **Limitation:** Being text-based, GeoJSON is inefficient for storing large datasets, as file sizes can quickly become unwieldy.

---

## 4. GeoTIFF and Cloud-Optimized GeoTIFF (COG)
Rasters require formats optimized for storing multi-band matrices.

* **GeoTIFF:** A standard TIFF image file with georeferencing metadata tags embedded (e.g., coordinate tie points, pixel scale, and projection).

* **Cloud-Optimized GeoTIFF (COG):** A modern extension of the GeoTIFF format designed for cloud storage.

  * *Internal Tiling:* The image is organized into small internal tiles ($256 	imes 256$ pixels), rather than continuous horizontal strips.

  * *Overviews (Pyramids):* Lower-resolution versions of the image are pre-rendered and stored within the same file.

  * *HTTP Range Requests:* When a web browser requests a map view, it only downloads the specific tiles and resolution level needed for that view, rather than downloading the entire multi-gigabyte file.

---

## 5. CSV (Comma-Separated Values) with Coordinates

* **Structure:** A plain text table where each row represents a point, and columns define the $X$ (longitude) and $Y$ (latitude) coordinates.

* **Usage:** Commonly used for exporting GPS logs, river gauge locations, and meteorological records.

* **GIS Import:** QGIS includes a **Delimited Text Provider** tool to parse CSV files and convert coordinate columns into vector point layers.

---

## 6. Modern Scientific & Multi-Dimensional Raster Formats
Traditional rasters represent static, 2D images. However, climate modeling, weather forecasting, and long-term hydrological observations require formats that support multi-dimensional and time-series variables:

* **NetCDF (Network Common Data Form - `.nc`):**
    * *Structure:* A self-documenting, machine-independent format designed for storing array-oriented scientific variables, dimensions (e.g., longitude, latitude, depth, time), and metadata.
    * *Hydrological Application:* Standard for multi-temporal climate and hydrology datasets (e.g., CHIRPS daily precipitation grids, global circulation model outputs, or ERA5 reanalysis). It allows analysts to store daily rainfall grids for an entire decade inside a single file.

* **HDF5 (Hierarchical Data Format 5 - `.h5`):**
    * *Structure:* A file format designed to store massive, complex, and heterogeneous datasets in a directory-like structure.
    * *Hydrological Application:* Used heavily by space agencies for remote sensing missions (e.g., GPM precipitation models, SMAP soil moisture profiles, and MODIS snow cover indices).

* **Zarr:**
    * *Structure:* Stores chunked, compressed, N-dimensional arrays in cloud object storage, allowing fast parallel access.
    * *Hydrological Application:* The modern, cloud-native successor to NetCDF. It enables distributed computing frameworks (like Dask) to process petabytes of climate and streamflow projections concurrently in the cloud.

---

## 7. Columnar & Binary Vector Formats
Traditional shapefiles and GeoJSON are slow to query and parse when dealing with massive datasets (e.g., global river networks). Columnar formats solve this by structuring data by column rather than row:

* **GeoParquet (`.parquet`):**
    * *Structure:* An extension of Apache Parquet (a columnar storage format) that adds spatial metadata and native geometry storage.
    * *Hydrological Application:* Ideal for massive spatial vector tables (e.g., millions of watershed boundaries). Because it is stored column-wise, a query can calculate the average area of sub-basins without scanning the entire file, resulting in queries that are 100x faster than traditional databases.

* **FlatGeobuf (`.fgb`):**
    * *Structure:* A binary vector format that includes a spatial index (R-Tree) packed inside the file.
    * *Hydrological Application:* Highly optimized for cloud streaming and web maps. It allows clients to fetch and render only the specific river segments within their active screen boundary, completely eliminating the need to download the full vector layer.

---

## 8. Point Cloud Formats (LiDAR & UAV Surveys)
High-resolution terrain surveys (e.g., for channel cross-sections, floodplain mapping, and landslide analysis) generate massive point clouds rather than grid arrays:

* **LAS (Laser File Format - `.las`):**
    * *Structure:* An open binary format maintained by the ASPRS for storing 3D point cloud data. It records $X,Y,Z$ coordinates, intensity, return number, scan angle, GPS time, and point classification (ground, water, high vegetation, buildings).
    * *Hydrological Application:* Base format for processing raw airborne LiDAR data. Ground points are extracted to generate Bare-Earth Digital Terrain Models (DTMs).

* **LAZ (`.laz`):**
    * *Structure:* A lossless compression of the LAS format, reducing file sizes by 70% to 90%. It is the standard format for sharing LiDAR data.

* **Cloud Optimized Point Cloud (COPC - `.copc.laz`):**
    * *Structure:* A standard LAZ file organized internally as a spatial octree.
    * *Hydrological Application:* Allows users to stream and view 3D point clouds of river beds dynamically in QGIS directly from cloud buckets, fetching only the points needed for the user's active view.

---

## 9. 3D & Mesh Formats
Hydraulic routing models (e.g., simulating wave propagation through rivers) use specialized meshes rather than standard rasters or vectors:

* **HEC-RAS HDF5 Meshes:**
    * *Structure:* Embedded HDF5 files containing structured or unstructured computational mesh geometries.
    * *Hydrological Application:* Used in HEC-RAS 2D hydraulic models to simulate river water surface elevations, flow velocities, and shear stress across complex floodplains.

* **3D Tiles:**
    * *Structure:* An open OGC standard designed for streaming massive 3D geospatial content (terrain, 3D buildings, meshes).
    * *Hydrological Application:* Streaming 3D mesh models of dams, river gorges, or urban catchment structures on web platforms.
