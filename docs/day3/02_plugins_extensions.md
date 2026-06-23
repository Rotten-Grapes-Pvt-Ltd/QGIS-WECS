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

### OSMDownloader

**Link:** [https://plugins.qgis.org/plugins/OSMDownloader/](https://plugins.qgis.org/plugins/OSMDownloader/)

Downloads OpenStreetMap vector data (roads, buildings, waterways) for a selected area of interest directly into QGIS.

* **Usage:** Indispensable for gathering baseline reference layers for your watershed modeling.

* **Execution:** Use the OSMDownloader tool to drag a rectangle on your map canvas and download all OSM vectors within the bounding box.

### qgis2web

**Link:** [https://plugins.qgis.org/plugins/qgis2web/](https://plugins.qgis.org/plugins/qgis2web/)

Exports a QGIS project to an interactive web map (using Leaflet or OpenLayers) with no coding required.

* **Usage:** Excellent for sharing your water maps, basin layouts, and flood extents with stakeholders as interactive web pages.

* **Execution:** Run the plugin, configure layers and zoom limits, and click export to generate a self-contained web map directory.

### AI Segmentation by TerraLab

**Link:** [https://plugins.qgis.org/plugins/terralab-ai/](https://plugins.qgis.org/plugins/terralab-ai/)

Uses AI models (like Meta's Segment Anything Model - SAM) to perform click-based vector segmentation on satellite or aerial imagery.

* **Usage:** Allows rapid digitization of surface water bodies, crop fields, or rooftops from high-resolution rasters.

* **Execution:** Select the model, click on a visual feature (such as a lake or forest outline) in your imagery, and the plugin automatically draws a matching vector boundary polygon.

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

---

## 4. Troubleshooting Common Plugin Errors

When installing or loading plugins, users frequently encounter path or package extraction errors.

### Error: "Couldn't load plugin '__MACOSX/qg'..."

* **The Cause:** This error occurs when a python plugin zip file was compiled on macOS and then manually extracted into the QGIS plugins directory. macOS automatically creates hidden metadata directories named `__MACOSX`. When QGIS scans the plugins directory, it incorrectly attempts to parse the `__MACOSX` folder as a python plugin. Since it lacks the required plugin files (like `__init__.py` and `metadata.txt`), QGIS throws an error.

* **The Solution:** 
  
  1. Open QGIS and navigate to **Settings** > **User Profiles** > **Open Active Profile Folder**.
  
  2. Navigate into the `python/plugins/` directory.
  
  3. Locate the hidden `__MACOSX` folder and **delete it** entirely.
  
  4. Restart QGIS, and the error warning will be resolved. Always use the built-in plugin manager to install zip files (via **Install from ZIP**) instead of manual extraction to avoid this issue.
