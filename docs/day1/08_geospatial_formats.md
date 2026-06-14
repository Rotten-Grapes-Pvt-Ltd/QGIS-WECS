# Geospatial Data Formats

Geospatial files must store coordinate geometries alongside descriptive attributes. Different formats have been optimized for different types of spatial analysis, data storage, and web sharing. This section explains the limitations of legacy shapefiles and the advantages of modern formats like GeoPackage, GeoJSON, and Cloud-Optimized GeoTIFF.

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
