# Accessing Open Satellite Data

Most satellite data is free and open to the public. This section details the primary portals used to download raw remote sensing datasets.

---

## 1. Copernicus Data Space Ecosystem (CDSE)
The **Copernicus Data Space Ecosystem (CDSE)** is the primary portal for European Space Agency (ESA) satellite datasets.

* **URL:** [dataspace.copernicus.eu](https://dataspace.copernicus.eu/)

* **Available Data:** Sentinel-1 (SAR), Sentinel-2 (Multispectral), Sentinel-3, and Copernicus DEM.

* **Search Tools:** Allows filtering by geographic area (drawing polygons), cloud cover threshold, acquisition date, and sensor type.

* **API Access:** Supports script-based downloading via OData and SpatioTemporal Asset Catalog (STAC) protocols.

---

## 2. USGS EarthExplorer
The primary portal maintained by the United States Geological Survey.

* **URL:** [earthexplorer.usgs.gov](https://earthexplorer.usgs.gov/)

* **Available Data:** Landsat (1-9), SRTM DEM, ASTER DEM, and global land cover products.

* **Search Tools:** Allows searching by address, coordinate list, or Shapefile upload.

* **Bulk Download:** Supports downloading multiple scenes using the USGS Bulk Download Application.

---

## 3. NASA Earthdata
The portal for NASA's Earth Observing System Data and Information System (EOSDIS).

* **URL:** [search.earthdata.nasa.gov](https://search.earthdata.nasa.gov/)

* **Available Data:** MODIS products (Evapotranspiration, Snow Cover, LST), GPM precipitation data, and soil moisture grids (SMAP).

* **Usage:** Best for downloading large-scale, daily global climate datasets.

---

## 4. Open Data Policies and Citations
When using satellite datasets in public commission reports (such as WECS studies), it is essential to follow licensing and citation guidelines:

* **Copernicus Data:** Free for reuse under Creative Commons licenses. It should be cited as: *"Contains modified Copernicus Sentinel data `[Year]`, processed by `[Author Name]`."*

* **USGS/NASA Data:** Public domain. Cite the specific mission and acquisition year (e.g., *"USGS Landsat 8 image courtesy of the U.S. Geological Survey, `[Year]`."*).
