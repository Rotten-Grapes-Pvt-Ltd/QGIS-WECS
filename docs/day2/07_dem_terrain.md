# DEMs and Terrain Data

Digital Elevation Models (DEMs) are the core datasets used in spatial hydrology. They define slopes, flow directions, and drainage pathways. This section explains DEM concepts, terrain derivatives, global datasets, and terrain data limitations in mountainous catchments.

!!! tip "Presentation Slides"
    You can download or view the lecture slides for this topic: [07_DEM_Terrain.pdf](presentations/07_DEM_Terrain.pdf)

---

## 1. DSM vs. DTM in Hydrology

Understanding the difference between elevation model types is critical for water resources modeling:

```text
       DSM (Digital Surface Model) - Captures treetops and building roofs
       ========================     ============
                               \   /            \
       DTM (Digital Terrain Model) - Captures bare ground surface
       _________________________\_/______________\_
```

* **Digital Terrain Model (DTM):** Represents the bare Earth terrain, filtering out buildings, trees, and other non-ground features. 
  
  * *Hydrological Standard:* Always use a DTM for rainfall-runoff, flow routing, and catchment delineation. Water flows along the physical ground, not over tree canopies or building roofs.

* **Digital Surface Model (DSM):** Represents the elevation of the top surface of all features, including buildings and vegetation canopy.
  
  * *Use Case:* Useful for urban stormwater modeling (direct runoff from rooftops) and calculating solar potential or line-of-sight viewsheds. Using a DSM in mountain catchments causes flow routing failures because forest canopies and bridges act as artificial dams/walls, blocking stream paths.

---

## 2. Primary Terrain Derivatives

Primary terrain attributes are calculated directly from a DEM grid by analyzing a moving $3 \times 3$ window of neighboring cells:

* **Slope:** The rate of change of elevation, measured in degrees ($0^{\circ}$ to $90^{\circ}$) or percentage rise ($0\%$ to $\infty$). Slope determines overland water velocity, soil erosive capacity, and hydrograph peak timing.

* **Aspect:** The compass direction that a slope faces ($0^{\circ}$ to $360^{\circ}$). In the Himalayas, aspect plays a vital role; south-facing slopes receive higher solar radiation, causing faster snowpack melting and elevated evapotranspiration compared to shaded, north-facing slopes.

* **Hillshade:** A shaded relief visualization created by simulating the sun's position (altitude and azimuth) in the sky. For general mapping, standard settings are $45^{\circ}$ altitude and $315^{\circ}$ azimuth (illumination from the northwest).

* **Curvature:** The rate of change of slope.
  
  * *Profile Curvature:* Parallel to the direction of the slope. It controls flow acceleration and deceleration along the flow path.
  
  * *Plan Curvature:* Perpendicular to the direction of the slope. It controls flow convergence (channeling) and divergence (spreading) across the landscape.

---

## 3. Compound Secondary Derivatives

Compound terrain attributes combine multiple primary attributes to model spatial processes:

* **Topographic Wetness Index (TWI):** Represents the spatial distribution of soil moisture saturation zones. It is calculated by balancing the upslope contributing area (which acts as a water source) and the local slope (which determines how fast water drains).
  
  * *Application:* Flat valley bottoms have high TWI values (slow drainage, high water collection), indicating potential wetlands or groundwater recharge zones. Steep ridges have low TWI values.

* **Stream Power Index (SPI):** Measures the erosive power of flowing water. It is calculated by combining the upslope contributing area and the local slope angle.
  
  * *Application:* Useful for identifying channel locations prone to severe gullying and bank erosion during flood events.

---

## 4. Global Elevation Datasets

| Dataset | Resolution | Source | Core Hydrological Utility / Characteristics |
| :--- | :--- | :--- | :--- |
| **Copernicus DEM (GLO-30)** | $30\text{ m}$ | European Space Agency (ESA) | The modern standard for regional basin modeling. High vertical accuracy, fully void-filled, and edited to correct water body boundaries. |
| **SRTM GL1** | $30\text{ m}$ | NASA / USGS | Historically popular radar dataset. Excellent for low-relief basins, but contains data voids in steep mountains and suffers from canopy bias. |
| **ALOS AW3D30** | $30\text{ m}$ | JAXA (Japan) | Generated from optical stereo imagery. Captures high morphological detail in steep, barren mountain peaks. |
| **Airborne LiDAR DTM** | $&lt; 1\text{ m}$ | Various | Airborne laser scanning that records multiple signal returns. Ground-return pulses penetrate forest canopy to create true bare-earth DTMs. Critical for local floodplain mapping. |
| **Bathymetric LiDAR** | $&lt; 2\text{ m}$ | Various | Uses green light lasers ($532\text{ nm}$) that penetrate clear water bodies to map channel beds and underwater bathymetry. |

---

## 5. DEM Limitations in Mountainous Terrain

In steep mountain environments (such as the Himalayas), global DEMs face significant structural errors:

* **Radar Layover & Shadow:** Steep gorges block side-looking radar pulses. This results in data shadows (missing values/voids) and distortions along steep slopes facing the sensor.

* **Forest Canopy Bias:** Spaceborne radar (C-band SRTM) and optical stereo matching (ALOS) cannot penetrate dense forest canopies. The elevation values returned represent the forest canopy top, not the bare earth, creating artificial mounds and filling in narrow river channels.

* **Sinks and Depressions:** Data noise and grid interpolation errors create artificial depressions (sinks) in the DEM where water flow paths terminate. Hydrologists must run a **Sink Filling** preprocessing step (using algorithms like Wang & Liu) to remove these errors and route flow continuously to the basin outlet. 
  
  * *Note:* Real endorheic lakes and karst depressions do exist, and sink-filling algorithms must be configured carefully to avoid erasing these features.

---

## 6. Guided Class Exercises

### Exercise 1: DTM vs. DSM routing through a forested valley
A hydrologist is setting up a rainfall-runoff model for a steep catchment. They use a Digital Surface Model (DSM) derived from satellite photogrammetry. The river flows through a dense forest canopy in a narrow canyon and passes under a highway bridge. 

When the model runs, it routes water away from the canyon, creating a massive artificial lake behind the bridge.

1. Identify the physical causes of this flow routing failure.

2. Explain how to resolve this problem using bare-earth DTM extraction or manual hydrological editing.

??? check "Answer Key - Exercise 1"

    1. **Physical Causes of Failure:**
    
        * **Canopy Blockage:** The DSM represents the top of the forest canopy. Because the canyon is densely forested, the DSM records a high elevation bridge of trees blocking the canyon floor. The routing algorithm sees this canopy as an artificial wall and diverts the water.
        
        * **Bridge Obstruction:** The DSM captures the highway bridge deck elevation, not the river channel flowing beneath it. This creates a solid block across the stream corridor, acting as an artificial dam.
        
    2. **Proposed Resolutions:**
    
        * **Transition to DTM:** The hydrologist should replace the DSM with a bare-earth Digital Terrain Model (DTM) where vegetation canopy and man-made structures have been filtered out.
        
        * **Hydrological Burning (Carving):** If a DTM is unavailable, the analyst can manually edit the DSM. In QGIS, they can draw a vector line along the true river channel, rasterize it, and subtract elevation values from the DSM along the line (a process called "stream burning" or "carving"). This cuts a channel through the bridge deck and tree canopy, allowing the model to route water correctly.

---

### Exercise 2: Utilizing TWI and managing shadow sinks
An analyst uses the Topographic Wetness Index (TWI) to map potential groundwater recharge zones in a rugged catchment. In the output map, they notice that several extremely high TWI values (indicating high wetness/saturation) are mapped on steep mountain slopes and narrow cliff gorges where soils are known to be dry and gravelly.

1. Explain the math-logical cause of these anomalously high TWI values in steep, rugged locations.

2. Detail the preprocessing step required to eliminate these errors before calculating TWI.

??? check "Answer Key - Exercise 2"

    1. **Math-Logical Cause of Anomalies:**
    
        * The TWI formula balances upslope contributing area ($\alpha$) and slope angle ($\beta$). Flat zones or depressions where flow accumulates have high contributing areas and low slope angles, resulting in high TWI values.
        
        * In rugged terrain, radar shadows or interpolation errors create artificial sinks (depressions) in the DEM. The flow routing algorithm routes all surrounding water into these closed depressions, artificially inflating the upslope contributing area ($\alpha$). 
        
        * Because water cannot exit the sink, it acts as a local drainage trap. The combination of an artificially massive contributing area and a flat depression floor produces anomalously high TWI values in dry, steep mountain gaps.
        
    2. **Required Preprocessing Step:**
    
        * **Sink Filling / Dem Sinks Removal:** Before calculating flow direction, flow accumulation, or TWI, the analyst must run a sink-filling tool (such as **Fill Sinks (Wang & Liu)** in the SAGA Processing Toolbox).
        
        * This tool identifies all closed depressions and raises their elevation cells to the level of the lowest adjacent outlet, allowing water to flow continuously downstream. This removes the local accumulation traps, correcting the contributing area calculation and eliminating the false high TWI anomalies on steep slopes.
