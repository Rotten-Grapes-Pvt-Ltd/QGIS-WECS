# Understanding Geospatial Datasets

Hydrological modeling requires importing and combining spatial datasets from multiple sources. This section details the core datasets used in water resource management, their file structures, and their applications in hydrological modeling.

---

## 1. Administrative Boundaries
Administrative boundaries define the borders of countries, provinces, districts, and municipalities.

* **Format:** Almost always represented as **Vector Polygons**.

* **Source:** National mapping agencies, geoportals, or global databases (e.g., GADM).

* **Hydrological Role:** While water flow is determined by topographic divides rather than administrative boundaries, these layers are critical for reporting statistics (e.g., calculating the total water demand or average rainfall per district).

* **GIS Best Practice:** Ensure topology rules are active to prevent gaps or overlaps between adjacent administrative divisions.

---

## 2. Drainage Networks and River Channels
River channels and stream networks model flow paths across the landscape.

* **Format:** **Vector Lines (Polylines)**.

* **Hydrological Role:** Used for route tracking, calculating travel times, and modeling stream channels in 1D hydraulic models.

* **Digital Stream Burning (Hydrological Conditioning):** When overlaying vector streams on top of digital elevation grids, the streams do not always align with the lowest points of the terrain due to elevation errors. Hydrologists use vector streams to "burn" (lower) the elevation values of the DEM along the channel, forcing the computer-calculated flow path to match the real-world river alignment.

---

## 3. Digital Elevation Models (DEMs)
A DEM represents the terrain elevation above a datum (like sea level).

* **Format:** Single-band **Raster Grid** (Float32).

* **DEM Subtypes:**

  * *Digital Terrain Model (DTM):* Represents the bare earth surface, excluding buildings and trees. **Always use DTM for hydrological modeling.**

  * *Digital Surface Model (DSM):* Represents the top surface of all features, including tree canopies and building rooftops. Useful for urban flood modeling.

```text
 DSM:  ===Tree===   ===Building===
        |       |    |          |
 DTM:  _Land____|____|__________|_  <-- Bare Earth Surface (Hydrology DEM)
```

* **Common Elevation Sources:**

  * **SRTM (Shuttle Radar Topography Mission):** $30\text{ m}$ global resolution. Reliable but can have voids in mountainous areas.

  * **ALOS PALSAR:** $12.5\text{ m}$ high-resolution terrain product. Very popular for slope analysis.

  * **Copernicus DEM (CopDEM):** $30\text{ m}$ modern global standard with high vertical accuracy.

---

## 4. Satellite Imagery
Satellite imagery provides multi-spectral and radar snapshots of the earth.

* **Format:** Multi-band **Raster Grids**.

* **Optical Satellites (Sentinel-2, Landsat 8/9):** Measure solar reflection across various wavelengths (blue, green, red, near-infrared, shortwave-infrared). Used for mapping surface water, crop types, and sediment plumes.

* **SAR Satellites (Sentinel-1):** Active radar sensors that measure backscattered radio waves. Essential for mapping flood extents during cloud cover or at night.

---

## 5. Land Use and Land Cover (LULC)
LULC datasets classify the land surface into categories like forest, urban, agriculture, and water bodies.

* **Format:** Integer **Raster Grids** (where each number represents a class ID) or **Vector Polygons**.

* **Hydrological Role:** LULC determines how much rainfall is absorbed by the ground vs. how much runs off. It is used to assign **Curve Numbers (CN)** in the USDA Soil Conservation Service (SCS) runoff model:

| LULC Class | Hydrological Properties | Runoff Potential |
| :--- | :--- | :--- |
| **Forest** | High soil permeability, root absorption, high intercept. | Low |
| **Agriculture** | Moderate soil permeability, subject to seasonal bare soil. | Medium |
| **Urban (Concrete)** | Impermeable surfaces, high drainage connectivity. | Maximum |

---

## 6. Precipitation and Rainfall Datasets
Rainfall data provides the input for water balance and runoff models.

* **Point Gauge Datasets:** Daily or hourly measurements from weather stations. These provide high temporal accuracy but poor spatial coverage in remote mountain areas.

* **Satellite Precipitation Estimates:**

  * **CHIRPS (Climate Hazards Group InfraRed Precipitation with Station data):** $5\text{ km}$ resolution, optimized for long-term trend analysis.

  * **GPM (Global Precipitation Measurement):** High temporal resolution (half-hourly), useful for flood event modeling.

* **Gridded Interpolation:** Hydrologists use interpolation tools in GIS to convert point weather station data into continuous surfaces (e.g., PRISM or Kriging interpolations) for input into basin models.
