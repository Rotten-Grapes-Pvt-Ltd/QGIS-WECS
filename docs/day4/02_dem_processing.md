# DEM Processing and Conditioning

Raw digital elevation models contain coordinate anomalies, canopy noise, and vertical measurement errors. These artifacts must be resolved before running hydrological models. This section covers sink identification, conditioning algorithms (breaching and filling), and vector stream burning.

---

## 1. Sinks, Depressions, and the Flow Routing Problem

A **Sink** (or pit) is a pixel or group of pixels surrounded entirely by cells of higher elevation values. 

```text
    RAW ELEVATION GRID              FLOW ROUTING BLOCK
    +-----+-----+-----+             +-----+-----+-----+
    | 120 | 118 | 122 |             |  v  |  v  |  v  |
    +-----+-----+-----+             +-----+-----+-----+
    | 121 | 110 | 119 |  ---------> |  >  | TERMINATE | <-- Flow path gets trapped
    +-----+-----+-----+             +-----+-----+-----+
    | 123 | 115 | 120 |             |  ^  |  ^  |  ^  |
    +-----+-----+-----+             +-----+-----+-----+
    [110m cell is a sink]
```

When a flow direction algorithm processes a raw DEM, water paths flow downhill until they hit a sink. Because all surrounding pixels are higher, the flow route terminates, creating a dry pool boundary. If these sinks are not resolved:

* Downstream flow accumulation grid calculations will drop to zero.

* The calculated river network will split into disconnected segments.

* Automatic watershed boundary delineations will fail or map tiny sub-basins.

While natural closed depressions exist (e.g., karst sinkholes, volcanic craters, or dry desert playas), the vast majority of sinks in $30\text{ m}$ global DEMs are measurement noise or canopy interpolation errors.

---

## 2. DEM Conditioning Algorithms

To enable continuous flow routing, QGIS Processing Toolbox includes algorithms from SAGA and GRASS to condition elevation grids:

### Wang and Liu Sink Filling (SAGA)

An advanced algorithm that identifies all depressions in the DEM and computes the spill elevation path (the lowest threshold coordinate along the depression's boundary). It raises the elevation of all cells within the sink to match the spill path, guaranteeing a continuous downhill path to the grid boundary.

### Planchon and Darboux Sink Filling (GDAL/GRASS)

Runs a virtual flood simulation across the entire DEM. It sets the elevation of all non-boundary cells to a high value and then slowly lowers their values from the edge inward, preserving the natural drainage pathways while raising depressions.

### Breaching (Sliver Cutting)

Instead of raising depression elevations (which can flatten valleys), breaching lowers the elevation of cells blocking the outlet path, carving a narrow trench through the barrier. Breaching is preferred in high-relief valleys to preserve steep slope profiles.

---

## 3. Stream Burning (The AGREE Algorithm)

Even after sink filling, computed flow paths can deviate from actual channels in flat plains or agricultural zones due to DEM cell resolution constraints. **Stream Burning** forces the hydrological routing engine to follow known, observed channels by lowering the DEM cells directly beneath a vector river network.

```text
    STREAM BURNING ELEVATION PROFILE (AGREE)
    
    DEM Elevation ----------------\                 /-----------------
                                   \  Smooth Buffer/
                                    \   /-----\   /
                                     \ /       \ /
                                      |  Sharp  |  <-- TRENCH BURNT
                                      |  Buffer |      (Forces flow path)
                                      +---------+
```

### The AGREE Algorithm (developed by UT Austin) runs a multi-step process:

1. **Sharp Buffer:** Generates a narrow buffer around the vector river lines (e.g., $10\text{ m}$) and drops the elevation values of cells inside this zone by a fixed height (e.g., $15\text{ meters}$).

2. **Smooth Buffer:** Generates a wider buffer (e.g., $100\text{ meters}$) and calculates a linear slope from the edges of this buffer down to the top of the sharp burnt trench, creating a smooth valley shape.

3. **Hydrological Routing:** Because a deep trench now exists in the DEM, the D8 flow routing engine is forced to route water along the actual river path, preventing flow line deviations.
