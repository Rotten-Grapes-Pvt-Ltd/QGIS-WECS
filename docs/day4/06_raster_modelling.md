# Raster-Based Hydrological Modelling

Raster datasets are used to model surface runoff, flow velocity, and soil erosion susceptibility.

---

## 1. Surface Runoff Modeling (Rational Method)
The **Rational Method** is a simple model used to calculate peak surface runoff discharge:

$$Q = C \times I \times A$$

Where:

* $Q$ = Peak runoff rate ($	ext{m}^3/	ext{s}$).

* $C$ = Runoff coefficient (derived by combining land use and soil type rasters).

* $I$ = Rainfall intensity ($	ext{mm}/	ext{hr}$).

* $A$ = Catchment area ($	ext{m}^2$).

---

## 2. Soil Erosion Susceptibility (RUSLE)
The **Revised Universal Soil Loss Equation (RUSLE)** is implemented in GIS to calculate annual soil loss rates:

$$A = R \times K \times LS \times C \times P$$

Where:

* $LS$ = Slope Length and Steepness factor (calculated from DEM).

* $C$ = Cover management factor (calculated from NDVI vegetation grids).
