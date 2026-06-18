# DEMs and Terrain Data

Digital Elevation Models (DEMs) are the core datasets used in spatial hydrology. They define slopes, flow directions, and drainage pathways. This section explains DEM concepts, terrain derivations, and terrain data limitations in mountainous areas.

---

## 1. DTM vs. DSM
Understanding the difference between elevation model types is critical:

```text
       DSM (Digital Surface Model) - Captures treetops and building roofs
       ========================     ============
                               \   /            \
       DTM (Digital Terrain Model) - Captures bare ground surface
       _________________________\_/______________\_
```

* **Digital Terrain Model (DTM):** Represents the bare Earth terrain, filtering out buildings, trees, and other non-ground features. **Always use a DTM for hydrological modeling**, as water flows along the physical ground, not over tree canopies.

* **Digital Surface Model (DSM):** Represents the elevation of the top surface of all features, including buildings and vegetation. Useful for urban runoff modeling and calculating line-of-sight viewsheds.

---

## 2. Terrain Derivatives
From a single elevation raster, GIS can calculate several secondary surfaces:

* **Slope:** The rate of change of elevation, measured in degrees ($0$ to $90$) or percentage ($0$ to $\infty$). Slope determines water velocity and runoff acceleration.

* **Aspect:** The compass direction that a slope faces ($0^{\circ}$ to $360^{\circ}$). Aspect determines solar exposure, which influences snowmelt timing and evaporation rates.

* **Hillshade:** A shaded relief visualization created by simulating the sun's position (altitude and azimuth) in the sky. This is used as a background layer to visualize terrain relief.

---

## 3. Global Elevation Datasets

| Dataset | Resolution | Agency | Hydrological Application |
| :--- | :--- | :--- | :--- |
| **Copernicus DEM** | $30\text{ m}$ | ESA | Modern standard for regional basin modeling. High vertical accuracy. |
| **SRTM GL1** | $30\text{ m}$ | NASA/USGS | Historically popular. Good for low-relief basins, but contains voids in high mountains. |
| **ALOS AW3D30** | $30\text{ m}$ | JAXA | Generated from optical stereo imagery. Excellent detail in steep slopes. |
| **LiDAR DEM** | $< 1\text{ m}$ | Various | High-precision engineering models, urban flood modeling, and river cross-sections. |

---

## 4. DEM Limitations in Mountainous Terrain
In steep mountain environments (such as the Himalayas), global DEMs have limitations:

* **Mountain Shadowing:** Radar signals can be blocked by steep peaks, resulting in data gaps (voids) in narrow valleys.

* **Canopy Bias:** In heavily forested catchments, C-band radar cannot fully penetrate the forest canopy, resulting in elevation values that are slightly higher than the actual bare ground.

* **Sinks and Depressions:** Vertical errors can create artificial depressions (sinks) in the DEM where water flow paths terminate. Hydrologists must run a **Sink Filling** preprocessing step to remove these errors before delineating watersheds.
