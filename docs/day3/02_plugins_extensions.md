# QGIS Plugins, Extensions, and Customization

QGIS features a highly modular structure, allowing users to extend its capabilities by installing community-developed plugins. These extensions range from simple basemap selectors to advanced image processing toolkits. This section covers QGIS plugin architecture, essential hydrology-relevant extensions, and PyQGIS scripting automations.

---

## 1. QGIS Plugin Architecture and Repository

Extensions are split into two groups:

* **Core Plugins:** Developed and maintained by the official QGIS team. They are written in C++ and are installed by default with QGIS (e.g., *DB Manager*, *Processing*, *Topology Checker*). They only need to be enabled in the settings.

* **External Python Plugins:** Contributed by community developers and hosted on the official QGIS Plugin Repository. They are written in Python and must be downloaded and installed within the QGIS interface.

### Installing Plugins:

1. Navigate to **Plugins** > **Manage and Install Plugins...**.

2. Under the **All** tab, type a search keyword.

3. Select the desired plugin and click **Install Plugin**. The tools are typically added to a new toolbar, panel, or context menu.

<video width="100%" controls>
  <source src="images/plugin.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

## 2. Essential Hydrology and Mapping Plugins

Several community plugins are standard requirements for water resources and terrain modeling:

### QuickMapServices (QMS)

**Link:** [https://plugins.qgis.org/plugins/quick_map_services/](https://plugins.qgis.org/plugins/quick_map_services/)

Allows you to add high-quality satellite and topographic background maps to the map canvas in one click.

* **Usage:** Provides visual context for digitized stream vectors and catchment boundaries.

* **Settings:** After installation, go to **Web** > **QuickMapServices** > **Settings** > **More services** and click **Get contributed pack** to unlock background maps from Google, Esri, Bing, and Mapbox.

### Profile Tool

**Link:** [https://plugins.qgis.org/plugins/profiletool/](https://plugins.qgis.org/plugins/profiletool/)

Generates cross-sectional elevation profiles of raster grids (DEMs) along a drawn vector line or river centerline.

* **Usage:** Indispensable for analyzing valley geometry, calculating river channel slopes, and identifying water flow obstructions.

* **Execution:** Add your DEM raster to the profile tool window, draw a line segment across the map canvas, and view the generated cross-sectional slope plot in real-time.

### Semi-Automatic Classification Plugin (SCP)

**Link:** [https://plugins.qgis.org/plugins/SemiAutomaticClassificationPlugin/](https://plugins.qgis.org/plugins/SemiAutomaticClassificationPlugin/)

A comprehensive remote sensing toolbox designed for land cover mapping.

* **Usage:** Features tools for downloading satellite datasets (Landsat, Sentinel), performing atmospheric corrections (DOS1), stacking bands, calculating spectral indices, and running supervised image classification algorithms.

---

## 3. Python Automation and PyQGIS Scripting

For repetitive workflows, QGIS exposes its entire graphical interface and geoprocessing API to Python via the **PyQGIS** library.

* **Accessing the Console:** Go to **Plugins** > **Python Console** (or press `Ctrl+Alt+P`).

```text
    +---------------------------------------------------------+
    |                    QGIS Python Console                  |
    +---------------------------------------------------------+
    >>> # Load vector layer programmatically
    >>> layer = iface.addVectorLayer("/path/to/rivers.gpkg", "Rivers", "ogr")
    >>> print(layer.featureCount())
    428
```

### Common PyQGIS Commands:

* **Load a Vector Layer:**
  `layer = iface.addVectorLayer("/data/vector/streams.gpkg", "River Network", "ogr")`

* **Load a Raster Layer:**
  `raster = iface.addRasterLayer("/data/raster/elevation.tif", "DEM Grid")`

* **Access the Active Layer:**
  `active_lyr = iface.activeLayer()`

* **Run a Geoprocessing Tool Programmatically:**
  ```python
  import processing
  
  # Run a buffer algorithm on the active layer and output to a file
  params = {
      'INPUT': active_lyr,
      'DISTANCE': 500,
      'SEGMENTS': 5,
      'END_CAP_STYLE': 0,
      'JOIN_STYLE': 0,
      'DISSOLVE': True,
      'OUTPUT': '/outputs/riparian_corridor.gpkg'
  }
  processing.run("native:buffer", params)
  ```
