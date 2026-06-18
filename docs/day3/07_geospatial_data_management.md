# Geospatial Data Management and Organization

Effective data management ensures that spatial datasets remain structured, documentable, and accessible across project lifetimes. Hydrological studies at WECS involve large vector river networks, multi-temporal raster grids, and external tables. This section covers directory organization, metadata documentation, and the transition from legacy Shapefiles to modern OGC GeoPackage standards.

---

## 1. Directory Structures and Versioning

Before initializing a GIS project, you must enforce a standard directory structure. Scattering data across temp directories or desktops leads to broken paths, lost datasets, and irreproducible analysis.

### Standard Workspace Folder Tree:

```text
    /Project_Root/
    ├── project_name.qgz       <-- Main QGIS project file (configured to Relative Paths)
    ├── data/
    │   ├── raw/
    │   │   ├── vector/        <-- Unmodified input boundaries, stations, and rivers
    │   │   └── raster/        <-- Raw satellite bands, climate grids, and original DEMs
    │   └── processed/
    │       ├── vector/        <-- Clipped, reprojected, or joined vector layers
    │       └── raster/        <-- Reclassified masks, slope surfaces, and filled DEMs
    ├── outputs/
    │   ├── maps/              <-- Exported PDF map layouts and high-res images
    │   └── reports/           <-- Extracted statistical summary tables (CSV/Excel)
    └── metadata/
        └── data_dictionary.md <-- Text catalog documenting data origins and parameters
```

---

## 2. Metadata Standards and Documentation

Metadata is "data about data." Without proper metadata, a dataset is a liability—its accuracy, age, and coordinate origin are unknown.

### Essential Metadata Parameters:

* **Data Source Lineage:** Who produced the dataset? (e.g., *USGS*, *Department of Hydrology and Meteorology - DHM*, *WECS*).

* **Acquisition Date/Time:** When was the data collected? Essential for evaluating seasonal effects in hydrology (monsoon vs. dry season).

* **Coordinate Reference System (CRS):** The exact EPSG reference code (e.g., `EPSG:32645` for UTM Zone 45N) used to register the coordinates.

* **Processing History:** A log of the alterations made to the raw data (e.g., *"Clipped to Bagmati watershed boundary, reprojected from EPSG:4326 using Bilinear resampling"*).

---

## 3. The Transition: Esri Shapefiles vs. OGC GeoPackages

For decades, the **Esri Shapefile** was the industry standard. However, it is an obsolete format designed in the early 1990s and has major technical limitations:

| Attribute | Legacy Esri Shapefile (`.shp`) | Modern OGC GeoPackage (`.gpkg`) |
| :--- | :--- | :--- |
| **File Architecture** | A collection of multiple separate files ($3$ to $15$ files, e.g., `.shp`, `.shx`, `.dbf`, `.prj`). | A single, self-contained SQLite database file (`.gpkg`). |
| **File Size Limit** | Hard limit of $2\text{ GB}$ per component file. | No hard size limit (scales to terabytes). |
| **Column Name Limit**| Column headers are strictly limited to $10\text{ characters}$ (attributes are truncated). | Unlimited column name lengths. |
| **Raster Support** | None. Vectors only. | Supports vector layers, raster grids, and non-spatial tables in a single file. |
| **Style Storage** | Requires separate `.qml` or `.sld` files. | Layer styles can be saved directly inside the GeoPackage database. |

### Why shapefiles crash in modern workflows:

* **File Drift:** If you copy a `.shp` file but forget the `.prj` projection file or the `.dbf` attribute file, the dataset becomes corrupted and unreadable.

* **No Database Constraints:** Shapefiles do not support database constraints, spatial indexes, or relational table links internally.

---

## 4. OGC GeoPackage Features and Benefits

The **OGC GeoPackage (GPKG)** is an open, standards-based, platform-independent container format. Behind the scenes, a GeoPackage is a standard SQLite database container containing specific system tables:

```text
    +-------------------------------------------------------+
    |                 GeoPackage File (.gpkg)               |
    +-------------------------------------------------------+
    |   [ System Tables ]      [ Vector Layers ]            |
    |   gpkg_contents,         ne_10m_rivers (Polylines),   |
    |   gpkg_spatial_ref_sys   catchment_boundary (Polygon) |
    +-------------------------------------------------------+
    |   [ Raster Layers ]      [ Non-Spatial Tables ]       |
    |   basin_dem (Raster),    monthly_rainfall_stats (CSV) |
    |   satellite_mask (Raster)                             |
    +-------------------------------------------------------+
```

### Key Advantages:

* **Transactional Safety:** GeoPackages support SQLite transactions. If a geoprocessing write operation is interrupted or crashes, the database rolls back, preventing file corruption.

* **Spatial Indexing:** Automatically builds spatial R-Tree indexes. This allows QGIS to query and render millions of vector shapes in milliseconds, whereas shapefiles require slow, sequential table scans.

* **Multi-Layer Collections:** You can store all vector catchments, stream lines, elevation rasters, and tabular climate gauge records for an entire watershed inside a single `catchment_database.gpkg` file, simplifying file transfer.
