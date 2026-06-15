# Web GIS and OGC Services

Online mapping services allow loading geographic data directly from remote servers without downloading the files locally. These services are governed by the **Open Geospatial Consortium (OGC)**.

---

## 1. OGC Web Service Types

* **WMS (Web Map Service):** Renders spatial data as a map image (e.g., PNG). Fast for background visualization, but does not allow querying raw coordinates or attributes.

* **WMTS (Web Map Tile Service):** Similar to WMS, but pre-renders maps into cached square tiles, speeding up pan and zoom operations in QGIS.

* **WFS (Web Feature Service):** Serves raw vector geometries and attributes. Allows users to query and download features.

* **WCS (Web Coverage Service):** Serves raw raster cell values (e.g., DEM elevations), making it suitable for running raster analysis.
