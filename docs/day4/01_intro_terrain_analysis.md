# Introduction to Terrain Analysis

Terrain analysis uses digital elevation datasets to analyze and model topographic surfaces. Topography is the primary driver of hydrological processes at the catchment scale, determining where water flows, how fast it travels, and where it gathers. This section details Digital Elevation Model (DEM) formats, global satellite elevation missions, and landform classification concepts.

---

## 1. Digital Elevation Model (DEM) Fundamentals

A DEM is a grid representation of the Earth's topographic elevation. It can be classified into three distinct categories depending on what features are captured:

```text
    ELEVATION MODEL TYPES
    +-------------------------------------------------------------+
    | DSM (Surface) : [Canopy]/\_/\_/\_/\_/\_/\_/\_/\_/\_/\_/\    |  <-- Captures tree tops, rooftops, terrain
    +-------------------------------------------------------------+
    | DTM (Terrain) : [Bare Earth]___________________________/\   |  <-- Captures ground surface only
    +-------------------------------------------------------------+
```

* **Digital Elevation Model (DEM):** A generic term for any digital representation of elevation.

* **Digital Surface Model (DSM):** Captures the absolute height of the first reflective surface, including tree canopies, buildings, power lines, and open ground. DSMs are generated directly from raw LiDAR or stereo satellite imagery.

* **Digital Terrain Model (DTM):** Represents the elevation of the bare Earth's surface, with vegetation, buildings, and other human-made structures mathematically filtered out. **DTMs are the standard requirement for hydrological routing**, as water flows along the physical ground, not over tree canopies.

### Grid Parameters and File Formats

* **Raster Format:** Typically stored as a single-band **Float32 GeoTIFF** file. Each pixel contains a decimal floating-point number representing height above a datum (e.g., $1450.72\text{ meters}$).

* **Coordinate Grid Alignment:** Raster cells are aligned in a grid defined by an origin coordinate ($X_{min}, Y_{max}$), cell dimensions (resolution), and spatial coordinates.

---

## 2. Comparison of Global Elevation Datasets

Selecting the right DEM is critical for modeling accuracy. Different datasets offer varying spatial resolutions and vertical accuracy limits:

| Dataset | Provider | Spatial Resolution | Vertical Accuracy | Hydrological Applicability |
| :--- | :--- | :--- | :--- | :--- |
| **SRTM GL1** | NASA / USGS | $30\text{ m}$ | $\approx 16\text{ m}$ | Historical baseline. Poor in high-slope mountain zones; contains voids (missing values) in steep Himalayan valleys. |
| **ALOS AW3D30** | JAXA | $30\text{ m}$ | $\approx 5\text{ m}$ | Generated from optical stereo imagery. Captures high-relief mountain ridges clearly, but can have anomalies in cloud-prone zones. |
| **Copernicus DEM** | ESA | $30\text{ m}$ (GLO-30) | $< 2\text{ m}$ | **Modern global standard**. High vertical accuracy and consistency; voids are filled using multi-sensor interpolation. |
| **LiDAR DTM** | Various | $< 1\text{ m}$ | $< 0.15\text{ m}$ | High-precision engineering projects, urban flood modeling, and precise river cross-section mapping. |

---

## 3. Surface Modelling and Landform Concepts

Topographic terrain is composed of distinct morphological features that govern catchment processes:

* **Ridges (Drainage Divides):** Local topographic high points. Water falling on opposite sides of a ridge divide flows into separate river systems. These boundaries define the catchment extents.

* **Valleys (Channels):** Local topographic low points that gather and route runoff. Channels are characterized by high flow accumulation values.

* **Saddles (Passes):** Low points along a ridge line between two peaks, representing potential flow overflow points during extreme flood routing.

* **Depressions (Sinks/Pits):** Low points lower than all surrounding cells, preventing water from flowing outward. Except for natural karst sinkholes or dry lakes, most depressions in raw DEMs are interpolation errors.

* **Plains (Flats):** Flat terrain where slope approaches $0^{\circ}$. Water velocity slows down dramatically in these zones, encouraging infiltration, groundwater recharge, or flooding.
