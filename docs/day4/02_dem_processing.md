# DEM Processing and Conditioning

Raw DEMs contain vertical measurement errors and canopy features that must be corrected before executing hydrological analysis.

---

## 1. The Sink Problem
Sinks are pixels surrounded by cells of higher elevation. When running flow routing algorithms, flow paths terminate inside these artificial depressions, preventing the calculation of downstream stream networks.

```text
    Raw DEM Elevation Profile:         Conditioned (Filled) DEM Profile:
    
    90m    88m    90m                  90m    90m    90m
    +---+  +---+  +---+                +---+  +---+  +---+
    |   |  |   |  |   |                |   |  |   |  |   |  <-- Filled Sink
    |   |  |Sink| |   |                |   |  |   |  |   |      (Allows water flow)
    +---+  +---+  +---+                +---+  +---+  +---+
```

---

## 2. DEM Conditioning Workflows
To correct raw datasets:

* **Sink Filling:** Algorithms (such as the **Wang and Liu** or **Planchon and Darboux** methods in GRASS/SAGA) identify sinks and raise their elevation values to match the lowest neighboring pixel, ensuring continuous downhill flow.

* **Stream Burning (AGREE Algorithm):** Lowers elevation grid values along a known vector river path. This forces the computer-calculated flow lines to match the actual, observed river channels.
