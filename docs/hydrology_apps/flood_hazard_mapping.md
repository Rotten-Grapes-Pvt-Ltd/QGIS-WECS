# Flood Hazard Mapping

Flood hazard mapping delineates geographic areas susceptible to inundation during high-flow events of specific return periods. By combining topographic models with active satellite sensor observations, GIS plays a vital role in disaster management, hydraulic routing, and zoning.

---

## 1. Core Objectives

*   **Extract** active flood extents using radar satellite imagery during monsoon or heavy storm events.

*   **Model** low-lying areas vulnerable to river spills using topographic indexes (such as HAND).

*   **Map** flood hazards and assets at risk (vulnerability overlays).

---

## 2. Key GIS Inputs

*   **Digital Elevation Model (DEM):** Bare-earth DTM (e.g., LiDAR or corrected Copernicus DEM) to resolve floodplain micro-topography.

*   **Active Satellite Imagery:** Sentinel-1 Synthetic Aperture Radar (SAR) imagery, which can penetrate cloud cover during active storm events.

*   **Vulnerability Layers:** Infrastructure vectors (roads, buildings, agricultural fields).

---

## 3. SAR-Based Active Flood Extraction Workflow

Since optical satellites cannot capture the ground through storm cloud cover, Synthetic Aperture Radar (SAR) is the primary tool for active flood delineation. Calm water bodies act as specular reflectors, scattering radar pulses away from the satellite, which results in very low backscatter (dark pixels):

```text
    RADAR BACKSCATTER SIGNATURE
    +--------------------------------+
    | Satellite Radar Pulse          |
    |      \     / (Dry Soil:        |
    |       \   /   Rough diffuse)   |
    |        v v                     |
    |   ______/\________ (Land)      |
    |                                |
    |      \                         |
    |       \ (Water: Specular       |
    |        \ reflection - no return)
    |~~~~~~~~~v~~~~~~~~~~~~~~~~ (Water)
```

1.  **Pre-process SAR Imagery:** (Using Sentinel-1 GRD imagery in Sentinel Application Toolbox or Google Earth Engine):
    
    *   Apply orbit file corrections.
    
    *   Remove thermal noise.
    
    *   Perform speckle filtering to reduce image grain.
    
    *   Perform Range Doppler Terrain Correction to orthorectify the grid to a DEM.

2.  **Determine Backscatter Threshold:**
    
    *   Water typically exhibits VV or VH polarization backscatter values below $-14\text{ dB}$. Plot a histogram of the SAR backscatter values to isolate the bimodal trough separating water and dry land.

3.  **Execute Threshold Masking:**
    
    *   Generate a binary raster mask where:
        
        $$\text{Flood Cell} = \begin{cases} 1 & \text{if Backscatter } < -15\text{ dB} \\ 0 & \text{if Backscatter } \ge -15\text{ dB} \end{cases}$$

4.  **Eliminate False Positives:**
    
    *   Himalayan mountain slopes cast radar shadows that mimic low backscatter. Use the terrain Slope raster calculated from your DEM to mask out all pixels where slope $> 5^\circ$, isolating calm water in flat valleys.

5.  **Vectorize Inundation:**
    
    *   Run **Polygonize** on the binary mask, select class $1$ polygons, and save the result as `active_flood_extent.gpkg`.

---

## 4. Topographic Flood Modeling (The HAND Model)

The **Height Above Nearest Drainage (HAND)** model is a static topographic index mapping potential flooding:

*   **Methodology:**
    
    1.  Delineate the stream network from the DEM.
    
    2.  Calculate the elevation of every pixel relative to the nearest stream channel it drains into.
    
    3.  HAND maps land relative vertical height above the drainage channel.
    
    4.  *Interpretation:* A pixel with a HAND value $< 2\text{ meters}$ is highly vulnerable to a $2\text{m}$ river rise during bank overflow events, regardless of its distance from the river centerline.

---

## 5. Hydrological Significance & Risk Overlay

*   **Flood Risk Assessment:** Intersect `active_flood_extent` polygons with building footprint points and agricultural land cover polygons to identify structural damage zones and crops lost.

*   **Hydraulic Calibration:** Delineated flood boundaries are compared against simulated 1D/2D hydraulic model extents (from HEC-RAS or LISFLOOD-FP) to calibrate Manning's roughness coefficients ($n$) and discharge predictions.
