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

### Modern OGC API Standards

Traditional OGC web services (WMS, WFS, WCS) were designed in the early 2000s and rely heavily on complex XML schemas, SOAP protocols, and verbose requests. The new **OGC API** standards represent a complete modernization of geospatial web streaming, designed for the modern web:

* **RESTful Architecture:** Uses clean HTTP methods (`GET`, `POST`, `DELETE`) and intuitive URL pathways (e.g., `/collections/rivers/items/1`).

* **JSON/GeoJSON Payloads:** Transmits data using lightweight GeoJSON for features and JSON for metadata instead of heavy XML/GML wrappers.

* **OpenAPI Documentation:** The service capabilities are documented using standard OpenAPI specifications, making them developer-friendly and indexable by search engine crawlers.

The core modern OGC API standards include:

* **OGC API - Features (replaces WFS):** Streams vector attributes and geometry using GeoJSON.

* **OGC API - Tiles (replaces WMTS):** Streams map tile sets (both raster and vector tiles).

* **OGC API - Coverages (replaces WCS):** Streams continuous gridded raster data values.

* **OGC API - Environmental Data Retrieval (EDR):** Provides a simple API for querying weather, oceanographic, and environmental point-series datasets.

---

## 3. Connecting to OGC Services in QGIS (Step-by-Step Exercises)

Connecting to remote geospatial web services in QGIS is handled through the **Browser Panel**. The following exercises use active, public web services from the USGS, Esri, and OGC testbeds.

### Exercise 1: Connect to an Esri XYZ/WMTS Basemap (High-Res Satellite Imagery)
XYZ tile connections are the fastest way to add high-resolution satellite basemaps to your project.

1. In the QGIS **Browser Panel**, right-click **XYZ Tiles** and select **New Connection...**.
2. Set the parameters:
   * **Name:** `Esri World Imagery`
   * **URL:** `https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}`
   * **Max. Zoom Level:** `19`
3. Click **OK**. Expand the **XYZ Tiles** list, locate **Esri World Imagery**, and double-click to add it to your map canvas.

### Exercise 2: Connect to a Public WMS Server (USGS Topographic Map)
WMS streams pre-rendered topographic base maps, perfect for referencing boundaries.

1. In the Browser Panel, right-click **WMS/WMTS** and select **New Connection...**.
2. Set the parameters:
   * **Name:** `USGS National Topographic Map`
   * **URL:** `https://basemap.nationalmap.gov/arcgis/services/USGSTopo/MapServer/WMSServer`
3. Click **OK**.
4. Expand the **WMS/WMTS** tree, expand the **USGS National Topographic Map** connection, locate the `0` or `USGS Topo Map` layer, and drag it into the Layers panel. Zoom in to watch the topological details render dynamically.

### Exercise 3: Connect to a Public WFS Server (US Government Boundary Lines)
WFS allows you to stream actual vector boundaries (polygons) directly into your attribute table.

1. In the Browser Panel, right-click **WFS / OGC API Features** (or **WFS**) and select **New Connection...**.
2. Set the parameters:
   * **Name:** `USGS Governmental Boundaries`
   * **URL:** `https://carto.nationalmap.gov/arcgis/services/govunits/MapServer/WFSServer`
3. Click **OK**.
4. Expand the connection, select a boundary layer (e.g., `State_or_Territory`), and drag it onto the canvas.
5. Use the **Identify Features** tool or press `F6` to open the Attribute Table. Notice that you have full access to the actual table attributes and boundary coordinates streamed from the server.

### Exercise 4: Connect to a Modern OGC API - Features Service
OGC API - Features represents the REST/GeoJSON standard for vector feature streaming.

1. In the Browser Panel, locate **WFS / OGC API Features** (in older versions, right-click **OGC API Features**). Select **New Connection...**.
2. Set the parameters:
   * **Name:** `Pygeoapi Demo Server`
   * **URL:** `https://demo.pygeoapi.io/master` (a public testing server running Pygeoapi).
3. Click **OK**.
4. Expand the connection tree. You will see several Collections (e.g., `Lakes`, `Natural Earth Countries`).
5. Double-click the `Lakes` collection. The points or polygons will stream instantly as lightweight GeoJSON, showing up on your map canvas with full attributes.

---

## 4. Web Map Publishing Platforms

If you need to share your maps with colleagues or the public via interactive web browsers, several open-source publishing avenues are available:

* **QGIS Server:** A C++ implementation of QGIS that runs on web servers, utilizing your existing QGIS project files (`.qgs`/`.qgz`) to serve WMS/WFS/WCS layers directly.

* **GeoServer:** The industry-standard Java-based open-source server for publishing spatial data, supporting transactional WFS (WFS-T) for collaborative editing.

* **QGIS Cloud:** A third-party web-hosting service integrated directly into QGIS via a plugin, allowing one-click publishing of maps to web browsers.
