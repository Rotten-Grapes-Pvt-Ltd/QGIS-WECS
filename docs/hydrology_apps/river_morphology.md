# River Morphology and Erosion

River channels are dynamic features that change shape, migrate across floodplains, and erode banks over time due to sediment transport and hydraulic forces. GIS and multi-temporal satellite observations allow engineers and hydrologists to track historical migrations, calculate river sinuosity, and assess bank erosion risks.

---

## 1. Core Objectives

*   **Extract** historical river centerlines and water boundaries over multiple decades.

*   **Measure** channel sinuosity and curvature changes.

*   **Delineate** zones of historical channel migration.

*   **Identify** river banks vulnerable to lateral erosion.

---

## 2. Key GIS Inputs

*   **Multi-Temporal Imagery:** Historical Landsat (5, 7, 8, 9) and Sentinel-2 satellite imagery spanning 30+ years.

*   **High-Resolution Topography:** DEMs (LiDAR or 12m WorldDEM) to extract channel bed elevations, bank slope profiles, and valley axis lines.

---

## 3. Extracting and Analyzing Channel Sinuosity

Sinuosity is the degree to which a river meanders. It is calculated by comparing actual channel length to straight-line valley length:

$$SI = \frac{L_{\text{channel}}}{L_{\text{valley}}}$$

Where:

*   $L_{\text{channel}}$ = Stream length measured along the channel centerline (m).

*   $L_{\text{valley}}$ = Straight-line length along the general trend of the valley axis (m).

```text
    CHANNEL SINUOSITY SCHEMATIC
       ___                 ___
      /   \    Channel    /   \
     |     \  L_channel  /     |
    -o------\-----------/------o-  Valley L_valley (Straight)
             \___      /
                 \____/
```

*   **Classification:**
    
    *   $SI < 1.05$: **Straight** channel.
    
    *   $1.05 \le SI < 1.5$: **Sinuous** channel.
    
    *   $SI \ge 1.5$: **Meandering** channel.

### GIS Workflow to Calculate SI

1.  Delineate water indices (e.g. MNDWI) and threshold to create waterbody polygons.

2.  Convert water polygons to centerlines: Use the Processing Toolbox **Thinning (Skeletons)** tool or WBT **Stream Vectorization** to extract a single-line vector representation of the channel.

3.  Split centerlines into study reach segments.

4.  Calculate $L_{\text{channel}}$ by using the Field Calculator on the line segment geometries (`$length`).

5.  Draw a straight vector line representing the valley trend and calculate its length ($L_{\text{valley}}$). Divide the channel length by the valley length to compute the Sinuosity Index.

---

## 4. Channel Migration Mapping (Time-Series overlays)

To visualize where the river has migrated over time:

1.  Extract water masks for multiple years (e.g., $1990, 2005, 2020$) using MNDWI thresholding.

2.  Vectorize each water mask into polygon boundaries: `water_1990`, `water_2005`, and `water_2020`.

3.  Run **Symmetric Difference** or **Union** between the layers. The non-overlapping areas highlight zones of lateral migration, meander cutoff loops, and island deposition.

4.  Calculate the lateral migration rate (meters/year) by measuring the distance between centerlines divided by the time interval.

---

## 5. Hydrological & Engineering Significance

*   **Infrastructure Safety:** Mapping channel migration corridors prevents the construction of roads, bridges, and buildings in areas where the river is actively migrating.

*   **Bank Stabilization:** Identifying high-sinuosity bend loops locates zones where high shear stress is directed at outer river banks, helping plan riprap, gabion, or bio-engineering bank stabilization works.

*   **Reservoir Sedimentation:** Fast channel migration and bank erosion upstream deliver heavy sediment loads to downstream reservoirs, reducing their active storage capacity. Tracking channel erosion helps locate sediment source areas.
