# Introduction to Terrain Analysis

Terrain analysis uses digital elevation datasets to analyze and model topographic surfaces. Topography is the primary driver of hydrological processes at the catchment scale, determining where water flows, how fast it travels, and where it gathers.

---

## 1. Digital Elevation Model (DEM) Fundamentals
A DEM is a grid representation of the Earth's elevation.

* **Format:** Single-band **Raster Grid** (Float32).

* **Ground Spacing (Resolution):** Cell dimensions on the ground. A $30	ext{ m}$ pixel captures an average height across a $30	ext{ m} 	imes 30	ext{ m}$ square, which generalizes small terrain features.

* **Vertical Accuracy:** The vertical margin of error (e.g., Copernicus DEM has a vertical accuracy of $< 2	ext{ meters}$ in low-slope areas, making it highly reliable for hydrological applications).

---

## 2. Surface Modelling Concepts
The Earth's terrain is characterized by:

* **Ridges (Divides):** Local topographic high points that separate drainage basins.

* **Valleys (Channels):** Local topographic low points that gather and route runoff.

* **Plains (Flats):** Flat areas where water velocity slows down, encouraging ponding and infiltration.
