# Introduction to QGIS

QGIS is a user-friendly, open-source Geographic Information System (GIS) licensed under the GNU General Public License. This section guides you through the interface, configuration, and project settings.

---

## 1. QGIS Interface Panels and Toolbars
The QGIS interface is composed of five key areas:

* **Menu Bar:** Provides access to general operations (Project, Edit, View, Layer, Settings, Plugins, Vector, Raster, Database, Processing).

* **Toolbars:** Icons for quick access to tools (File, Map Navigation, Attributes, Label, Vector, Raster).

* **Layers Panel (Layer Tree):** Lists all datasets loaded in the project. The drawing order is top-down (layers at the top draw on top of layers below).

* **Browser Panel:** A built-in file explorer to browse local drives, databases (PostgreSQL/PostGIS, SpatiaLite, Geopackage), and web services (WMS/WFS/XYZ).

* **Map Canvas:** The central window where spatial data is rendered.

---

## 2. The Processing Toolbox
The **Processing Toolbox** is the command center for running spatial analysis.

* **Accessing:** Click **Processing** > **Toolbox** (or press `Ctrl+Alt+T`).

* **Search Provider:** You can search for tools by keyword. It includes native QGIS algorithms alongside external providers like **GRASS**, **SAGA**, and **GDAL**.

---

## 3. Project Management
A QGIS project file (`.qgz` or `.qgs`) is a text file written in XML format. It does **not** store the actual spatial data (like Shapefiles or GeoPackages). Instead, it stores:

* Relative or absolute file paths to the datasets.

* Layer styles, color ramps, and labels.

* Project CRS and coordinate settings.

* Print layouts and compositions.
