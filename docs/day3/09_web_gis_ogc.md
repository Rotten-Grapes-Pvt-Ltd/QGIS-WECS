# Web GIS and OGC Services

Online mapping services allow you to stream and query geospatial data directly from remote servers without downloading the physical files. These services are governed by the **Open Geospatial Consortium (OGC)**, ensuring interoperability across different GIS software clients. This section details OGC service classifications, their configuration inside QGIS, and web publishing pathways.

---

## 1. Web GIS Architecture and OGC Standards

Web GIS operates on a client-server model:

```text
    +------------------+                   +--------------------+
    |    QGIS Client   | --- REQUEST ----> |   Web GIS Server   |
    | (Browser/Map View) | <--- RESPONSE --- | (GeoServer/MapServer)|
    +------------------+                   +--------------------+
                           (JSON/XML/PNG)
```

The client (e.g., QGIS Desktop) sends a request containing coordinate bounds, projection information, and style parameters. The server (e.g., GeoServer, QGIS Server) processes the database and returns the requested data.

---

## 2. OGC Web Service Types

OGC has defined standard protocols for sharing different classes of geospatial information:

### WMS (Web Map Service)

Delivers pre-rendered map images (usually in PNG or JPEG format) corresponding to the client's screen view.

* **Characteristics:** Fast because the server does all the rendering. However, the client receives only a flat image; you cannot query the raw vector coordinates or change layer symbology styles.

* **Hydrological Application:** Accessing regional land cover maps or historical administrative boundaries as visual basemaps.

### WMTS (Web Map Tile Service)

An advanced version of WMS that serves maps using a pre-calculated pyramid of tile images.

* **Characteristics:** The server partitions the map into standard grid tiles (e.g., $256 \times 256\text{ pixels}$) across preset zoom levels. The client's browser caches these tiles, resulting in smooth panning and zooming.

* **Hydrological Application:** High-resolution satellite basemaps (e.g., Google Satellite, Esri Terrain).

### WFS (Web Feature Service)

Streams raw vector features (including coordinates and attribute tables) using XML/GML or GeoJSON formats.

* **Characteristics:** Allows the client to select features, run attribute queries, edit geometries, and save the data locally. WFS can be slow if the requested layer contains millions of complex coordinates.

* **Hydrological Application:** Streaming national rain gauge points or active river monitoring station locations.

### WCS (Web Coverage Service)

Streams raw, multi-dimensional raster cell values (e.g., actual elevation values from a DEM or rainfall numbers in $mm$).

* **Characteristics:** Unlike WMS (which only shows an image of a DEM), WCS sends the actual numbers. This allows the client to run geoprocessing algorithms (such as slope calculation or raster math) on the streamed dataset.

* **Hydrological Application:** Streaming regional DEMs or daily precipitation grids for runoff modeling.

---

## 3. Connecting to OGC Services in QGIS

Connecting to OGC servers requires a service endpoint URL:

1. Locate the service URL (e.g., Copernicus CDSE browser service endpoints or WECS registry server links).

2. In the QGIS **Browser Panel**, locate the service type:

   * Right-click **WMS/WMTS** to add an image stream.

   * Right-click **WFS** to add vector features.

   * Right-click **WCS** to add raw raster coverage.

3. Select **New Connection...**.

4. Enter a name for the connection and paste the server URL.

5. Click **OK**. Expand the connection listing in the Browser Panel, select the desired layer, and drag it onto the Map Canvas.

---

## 4. Web Map Publishing Platforms

If you need to share your maps with colleagues or the public via interactive web browsers, several open-source publishing avenues are available:

* **QGIS Server:** A C++ implementation of QGIS that runs on web servers, utilizing your existing QGIS project files (`.qgs`/`.qgz`) to serve WMS/WFS/WCS layers directly.

* **GeoServer:** The industry-standard Java-based open-source server for publishing spatial data, supporting transactional WFS (WFS-T) for collaborative editing.

* **QGIS Cloud:** A third-party web-hosting service integrated directly into QGIS via a plugin, allowing one-click publishing of maps to web browsers.
