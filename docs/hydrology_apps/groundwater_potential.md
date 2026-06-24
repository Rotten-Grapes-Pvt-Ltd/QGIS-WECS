# Groundwater Potential Mapping

Groundwater is a vital sub-surface water resource. Unlike surface water, aquifers are hidden. GIS enables the mapping of groundwater potential zones (GWPZ) by combining multiple thematic layers representing surface and sub-surface conditions that control infiltration and storage.

> [!TIP]
> **Data Sources & Acquisition:**
> If you lack geological, soil, or land cover data for groundwater potential zoning:
> 
> *   **Global Geology and Lithology Layers:** Download global-scale rock and geologic map layers from the [USGS Mineral Resources Program](https://mrdata.usgs.gov/geology/world/) or query regional maps from national geological portals (such as national mining and geology bureaus).
> 
> *   **Harmonized World Soil Database (HWSD):** Provides global soil textures, clay percentages, and hydraulic properties. Download the spatial database for free from the [IIASA HWSD Portal](https://gryphon.iiasa.ac.at/index.php/HWSD).
> 
> *   **Land Use / Land Cover (LULC) maps:** Download the $10\text{ m}$ global land cover classification grid from the [ESA WorldCover Portal](https://esa-worldcover.org/).

---

## 1. Core Objectives

*   **Compile** thematic factors controlling aquifer recharge (slope, geology, soil, lineaments).

*   **Evaluate** factor weights using Multi-Criteria Decision Analysis (MCDA) and the Analytic Hierarchy Process (AHP).

*   **Compute** a spatial weighted overlay to map groundwater potential zones.

---

## 2. Key Spatial Factors

Groundwater potential depends on surface runoff potential and sub-surface porosity:

*   **Lithology / Geology:** Permeable rocks (e.g., weathered sandstone, fractured limestone) permit infiltration, while crystalline granite blocks it.

*   **Slope:** Flat plains ($<2^\circ$) retain water, permitting high infiltration. Steep slopes ($>20^\circ$) encourage rapid surface runoff.
    *   *GIS Toolpaths:* **SAGA** > **Terrain Analysis - Morphometry** > **Slope, Aspect, Curvature** OR **WhiteboxTools** > **Geomorphometric Analysis** > **Slope**.

*   **Lineament Density:** Fault lines, fractures, and joints increase secondary porosity, creating storage pathways for groundwater.
    *   *GIS Toolpaths:* **SAGA** > **Grid - Tools** > **Kernel Density Estimation** OR **WhiteboxTools** > **Math and Stats Tools** > **LineDensity**.

*   **Soil Texture:** Coarse sandy soils have high hydraulic conductivity compared to tight clay soils.

*   **Land Use / Land Cover (LULC):** Vegetation (forests) slows runoff, increasing infiltration. Impervious urban concrete blocks recharge.

*   **Drainage Density:** High drainage densities represent high surface runoff, leaving less water for sub-surface recharge.
    *   *GIS Toolpaths:* **SAGA** > **Terrain Analysis - Channels** > **Drainage Density** OR **WhiteboxTools** > **Hydrological Analysis** > **DrainageDensity**.

---

## 3. The Analytic Hierarchy Process (AHP) Workflow

AHP is a mathematical method used to calculate factor weights based on pairwise comparisons:

1.  **Pairwise Comparison Matrix:** Compare factors in a matrix, scoring them from $1$ (equal importance) to $9$ (extreme importance).
    
    *   *Example:* Lithology is twice as important as Slope ($2$), and four times as important as LULC ($4$).

2.  **Calculate Eigenvector (Weights):** Normalize the matrix to find the priority vector (weights). The sum of all weights must equal $1.0$.

3.  **Verify Consistency:** Calculate the **Consistency Ratio (CR)**:
    
    $$CR = \frac{CI}{RI}$$
    
    Where $CI$ is the consistency index and $RI$ is the random consistency index.
    
    *   *Rule:* If $CR < 0.1$, the pairwise weights are logically consistent. If $CR \ge 0.1$, the comparison matrix must be re-evaluated.

---

## 4. GIS Weighted Overlay Workflow

Once weights are established, GIS is used to compile the final potential map:

```text
    THEMATIC FACTOR INPUTS              WEIGHTS         FINAL OVERLAY MAP
    +------------------------------+
    | Lithology (Reclassified 1-5) | x   0.35   \
    +------------------------------+             \
    | Slope     (Reclassified 1-5) | x   0.25     \     +----------------+
    +------------------------------+               ---> |  Groundwater   |
    | Lineaments(Reclassified 1-5) | x   0.20     /     | Potential Map  |
    +------------------------------+             /      +----------------+
    | Soil / Land (Reclassified 1-5)| x   0.20   /
    +------------------------------+
```

1.  **Standardize Grids:**
    
    *   Reproject all factor layers to a single projected CRS (e.g., UTM Zone 45N).
    
    *   Resample rasters to match the spatial resolution of the DEM (e.g., $30\text{m}$).

2.  **Reclassify Factors:**
    
    *   Run **Reclassify** to convert raw values into a standardized scale of $1$ (Very Low Recharge) to $5$ (Very High Recharge).
        
        *   *Slope reclassification:* $0-2^\circ \rightarrow 5$; $2-5^\circ \rightarrow 4$; $5-10^\circ \rightarrow 3$; $10-20^\circ \rightarrow 2$; $>20^\circ \rightarrow 1$.
        
    *   *GIS Toolpaths (Reclassify):*
        
        *   **SAGA GIS:** **Processing Toolbox** > **SAGA** > **Grid - Tools** > **Reclassify Grid Values** (or **Reclassify by Table**).
        
        *   **WhiteboxTools:** **Processing Toolbox** > **WhiteboxTools** > **Math and Stats Tools** > **Reclassify**.

3.  **Execute Weighted Overlay:**
    
    *   *Equation:*
        
        $$GWPZ = \sum_{i=1}^{n} (W_i \times C_i)$$
        
        $$\text{GWPZ} = (0.35 \times \text{Lithology}) + (0.25 \times \text{Slope}) + (0.20 \times \text{Lineament}) + (0.15 \times \text{Soil}) + (0.05 \times \text{LULC})$$
        
    *   *GIS Toolpaths (Weighted Overlay):*
        
        *   **SAGA GIS:** **Processing Toolbox** > **SAGA** > **Grid - Calculus** > **Grid Calculator** (best for complex customized overlay formulas).
        
        *   **WhiteboxTools:** **Processing Toolbox** > **WhiteboxTools** > **Math and Stats Tools** > **WeightedOverlay** (or **RasterCalculator**).

---

## 5. Hydrological Significance

*   **Borehole Siting:** Identifying areas marked as "High" or "Very High" groundwater potential increases the success rate of drilling clean drinking water wells.

*   **Artificial Recharge Management:** Locating check dams and recharge pits in "High" potential zones where soils and slopes are optimized for infiltration accelerates aquifer recovery.

*   **Aquifer Protection Zoning:** Overlaying high potential zones with agricultural pollution maps outlines regions where groundwater is vulnerable to chemical contamination.
