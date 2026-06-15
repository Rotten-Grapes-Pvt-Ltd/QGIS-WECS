# Earth Observation Missions and Sensors

To retrieve hydrological datasets, we must understand the specific satellite platforms orbiting the Earth. This section details the core spaceborne sensors used by WECS for watershed management, flood mapping, and climate trend analysis.

---

## 1. Sentinel-1 (European Space Agency - ESA)
Sentinel-1 is a constellation of two polar-orbiting satellites carrying active C-band Synthetic Aperture Radar (SAR) sensors.

* **Sensor Type:** Active Radar (C-band, wavelength $\approx 5.6	ext{ cm}$).

* **Spatial Resolution:** $10	ext{ m}$ (in Interferometric Wide swath mode).

* **Temporal Resolution:** 6 to 12 days revisit.

* **Polarizations:** VV, VH, HH, HV.

* **Hydrological Role:** Flood inundation mapping. Because C-band radar penetrates cloud cover, Sentinel-1 is the primary tool used to map flood boundaries during the active monsoon season.

---

## 2. Sentinel-2 (European Space Agency - ESA)
Sentinel-2 is a constellation of two optical satellites carrying the Multispectral Instrument (MSI).

* **Sensor Type:** Passive Optical (13 spectral bands).

* **Spatial Resolution:**

  * $10	ext{ m}$ (Blue, Green, Red, NIR bands).

  * $20	ext{ m}$ (Red Edge, SWIR bands).

  * $60	ext{ m}$ (Atmospheric correction bands).

* **Temporal Resolution:** 5 days revisit.

* **Hydrological Role:** Water body mapping, snow cover monitoring, vegetation health index generation (NDVI), and sediment monitoring.

---

## 3. Landsat 8 and 9 (NASA / USGS)
The Landsat program provides the longest continuous global record of the Earth's surface (since 1972). Landsat 8 and 9 carry the Operational Land Imager (OLI) and the Thermal Infrared Sensor (TIRS).

* **Sensor Type:** Passive Optical & Thermal.

* **Spatial Resolution:**

  * $15	ext{ m}$ (Panchromatic band).

  * $30	ext{ m}$ (Visible, NIR, SWIR bands).

  * $100	ext{ m}$ (Thermal bands, resampled to $30	ext{ m}$).

* **Temporal Resolution:** 8 to 16 days revisit.

* **Hydrological Role:** Historical watershed change detection, crop water requirement estimation, and reservoir surface temperature mapping.

---

## 4. MODIS (NASA Terra & Aqua)
The Moderate Resolution Imaging Spectroradiometer (MODIS) is designed for large-scale, daily environmental monitoring.

* **Sensor Type:** Passive Optical.

* **Spatial Resolution:** Coarse ($250	ext{ m}$, $500	ext{ m}$, $1000	ext{ m}$).

* **Temporal Resolution:** Daily revisit.

* **Hydrological Role:** Daily snow cover tracking, evapotranspiration modeling, and monitoring regional-scale water index anomalies.

---

## 5. SRTM (Shuttle Radar Topography Mission)
An international research effort that obtained elevation data on a global scale.

* **Sensor Type:** Radar Interferometry (C-band and X-band).

* **Acquisition Date:** February 2000.

* **Spatial Resolution:** $30	ext{ m}$ global grid.

* **Hydrological Role:** Baseline digital elevation dataset used for regional watershed boundary delineation and slope calculations.

---

## 6. CHIRPS (Climate Hazards Group InfraRed Precipitation)
A gridded, multi-decadal precipitation dataset.

* **Type:** Hybrid (combines satellite infrared estimates with point rain gauge data).

* **Spatial Resolution:** $0.05^{\circ}$ (approximately $5.5	ext{ km}$ grid).

* **Temporal Resolution:** Daily, pentad (5-day), and monthly records.

* **Hydrological Role:** Long-term precipitation trend analysis, basin-wide water budget calculations, and regional drought modeling in areas with sparse weather station coverage.
