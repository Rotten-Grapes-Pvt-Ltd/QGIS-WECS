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

### QGIS Project File Path Configuration (Relative vs. Absolute Paths)

When QGIS links to spatial datasets (e.g., GeoPackages, Shapefiles, or Rasters), it stores the file paths inside the `.qgz` project file. The path storage configuration determines whether you can share your project folder without breaking data connections:

* **Absolute Paths:** Stores the full system path (e.g., `/Users/krishnaglodha/Documents/work/wb/QGIS-WECS/docs/data/Natural_Earth_quick_start/DEM/output_hh.tif`). If you move the project folder to a different computer or share it with a colleague, all links will break, throwing a **Handle Bad Layers** error dialog.
* **Relative Paths (Recommended):** Stores paths relative to the position of the `.qgz` project file (e.g., `./docs/data/Natural_Earth_quick_start/DEM/output_hh.tif`). This allows you to copy, zip, or move the entire project root folder structure to any directory on any computer without breaking any database links.

#### Setting Relative Paths in QGIS:
1. Go to **Project** > **Properties...** (`Ctrl+Shift+P`).
2. Select the **General** tab.
3. Under **General settings**, find **Save paths**. Change this dropdown from **Absolute** to **Relative**.
4. Click **Apply** and **OK**. Save your project.

### QGIS Backup Files (`.qgz~`)

Whenever you save a project in QGIS, the software creates a backup file alongside your main file, appended with a tilde symbol (e.g., `project_name.qgz~` or `project_name.qgs~`). 

* **Purpose:** Acts as a recovery point in the event of a system crash, power loss, or database write interruption that corrupts your main `.qgz` project file.
* **How to Restore:** If your main project file fails to open, navigate to your workspace root directory. Delete the corrupted `project_name.qgz` file. Rename the backup file by stripping off the trailing tilde (rename `project_name.qgz~` to `project_name.qgz`). Open this renamed file in QGIS to recover your project up to the last successful save point.

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

---

## 5. Safe File Manipulation using the QGIS Browser Panel

A common mistake is attempting to rename, copy, or delete shapefiles or database files using your operating system's native file explorer (Windows Explorer or macOS Finder). Because GIS files (especially shapefiles) are composite formats made up of multiple linked files, this manual file manipulation frequently breaks reference links and corrupts datasets.

To safely organize your files, always use the built-in **QGIS Browser Panel**:

```text
    QGIS BROWSER PANEL ACTIONS
    +----------------------------------+
    | Browser                          |
    +----------------------------------+
    |  * Favorites  --> Add Shortcuts  |
    |  * Project    --> Active Files   |
    |  * GeoPackage --> [Right-Click]  |
    |                   ├── New GPKG...|
    |                   ├── Delete...  |
    |                   └── Rename...  |
    +----------------------------------+
```

* **Adding Folder Shortcuts:** Right-click **Favorites** in the Browser Panel and select **Add a Directory...**. Pin your main project root folder (`QGIS-WECS`) here to jump to your workspace folders instantly.
* **Creating a New Database:** Right-click **GeoPackage** in the Browser panel, select **Create Database...**, set the file destination, name the table, and configure the spatial parameters (CRS, geometry type).
* **Safe Table Operations:** To rename, copy, or delete a layer table inside a GeoPackage, right-click the table name inside the Browser panel and select the respective operation. QGIS will safely modify the SQL database schema without corrupting structural system files.

---

## 6. Inspecting GeoPackages with the DB Manager

Because the GeoPackage is a standard SQLite database container, you can interact with it using database commands. QGIS includes a built-in database client interface called the **DB Manager**.

* **Accessing:** Go to **Database** > **DB Manager...**.

Inside the DB Manager, you can inspect spatial databases without adding them to your Layers panel:

1. Expand the **GeoPackage** tree in the left pane of the DB Manager.
2. Select your target `.gpkg` database (e.g., `catchment_analysis.gpkg`) to view database parameters.
3. Select a specific table inside the database. The right pane provides three tabs:
   * **Info:** Displays structural details like table columns, primary key constraints, coordinate reference system, and index flags.
   * **Table:** Renders the tabular spreadsheet of attributes directly.
   * **Preview:** Renders a quick, non-interactive map rendering of the spatial features.
4. **SQL Window:** Click the SQL window icon in the toolbar (or press `F2`) to open a query editor. You can write custom SQL statements (e.g., `SELECT NAME, POP_EST FROM ne_10m_admin_0_countries WHERE CONTINENT = 'Asia'`) to query, filter, and extract layers directly from your database.

---

## 7. Consolidating Workspaces with the Package Layers Tool

At the end of a spatial analysis project, you may have layers loaded from multiple temporary directories, external network drives, or different folders. If you need to share this project with a colleague, packaging these scattered source layers is a tedious process.

QGIS provides the **Package Layers** tool to automate workspace consolidation:

1. Open the **Processing Toolbox** (`Ctrl+Alt+T`) and search for **Package Layers**.
2. **Input layers:** Click `...` and check the boxes for all vector layers you want to consolidate in your workspace.
3. **Destination GeoPackage:** Click `...` > **Save to File** and define the path for a new database (e.g., `final_project_archive.gpkg`).
4. Keep the boxes checked for **Save style configurations** (packages your symbology, colors, and label settings inside the GeoPackage) and **Save metadata** (retains dataset descriptors).
5. Click **Run**. QGIS compiles all separate vector layers, along with their active symbology properties, into a single `.gpkg` file. You can share this database container and the `.qgz` project file with any user, and it will render perfectly on their machine.
