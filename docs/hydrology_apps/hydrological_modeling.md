# Hydrological Modeling with HEC-HMS / SWAT

Hydrological models simulate the movement of water through a watershed, converting rainfall into river discharge. GIS is the primary tool used to pre-process spatial datasets (topography, soil, land use) and parameterize these complex models.

---

## 1. Core Objectives

*   **Partition** the watershed into uniform hydrologic subdivisions.

*   **Derive** physical model parameters (such as routing lengths and travel times).

*   **Generate** Soil Conservation Service (SCS) Curve Number grids.

*   **Initialize** Soil & Water Assessment Tool (SWAT) and HEC-HMS workspaces.

---

## 2. Key GIS Inputs

*   **Digital Elevation Model (DEM):** Used to delineate sub-basins, calculate reach lengths, slope profiles, and flow pathways.

*   **Land Use / Land Cover (LULC) Map:** Classified grid indicating surface cover (forest, urban, crop), controlling surface roughness and infiltration.

*   **Soil Map:** Thematic soil database containing physical characteristics (clay percentage, bulk density, water content) linked to Hydrological Soil Groups.

---

## 3. SWAT Pre-Processing: Hydrological Response Units (HRUs)

The **Soil & Water Assessment Tool (SWAT / QSWAT+)** is a continuous, semi-distributed model. Rather than modeling cell-by-cell, SWAT groups similar areas into **Hydrological Response Units (HRUs)**:

```text
    SWAT HRU DERIVATION
    +----------+     +----------+     +----------+     +--------------------+
    |  LULC    |  +  | Soil Map |  +  | Slope    |  =  | Hydrological       |
    | (Forest) |     | (Sandy)  |     | (Flat)   |     | Response Unit (HRU)|
    +----------+     +----------+     +----------+     +--------------------+
```

*   **HRU Definition:** An HRU is a unique combination of a specific land use, soil type, and slope class within a sub-basin.

*   **GIS Pre-Processing Workflow:**
    
    1.  Load the LULC and Soil vector/raster layers in QSWAT.
    
    2.  Run SAGA Slope on the DEM and group slopes into classes (e.g. $0-5\%$, $5-15\%$, $>15\%$).
    
    3.  Run the **HRU Definition** overlay tool. QSWAT intersects these three layers, eliminates fractional sliver polygons below a user threshold (e.g. $<5\%$ area), and generates a database file parameterizing the hydrology of each HRU.

---

## 4. HEC-HMS Pre-Processing: SCS Curve Number (CN) Delineation

**HEC-HMS** is a semi-distributed event-based model developed by the US Army Corps of Engineers. One of its primary loss models is the **SCS Curve Number (CN)** method, which estimates excess runoff depth based on soil and land cover:

$$Q = \frac{(P - I_a)^2}{P - I_a + S}$$

Where:

*   $Q$ = Direct runoff depth (mm).

*   $P$ = Precipitation depth (mm).

*   $I_a$ = Initial abstraction (surface storage, interception, infiltration before runoff, mm).

*   $S$ = Potential maximum retention after runoff begins (mm), derived from the Curve Number:
    
    $$S = \frac{25400}{CN} - 254$$

*   **Curve Number (CN) Range:** Ranges from $30$ (permeable forest soils, low runoff) to $100$ (completely impervious concrete surfaces, total runoff).

### GIS Workflow to Generate CN Grids

1.  Reclassify the LULC map to match standard SCS land use descriptions.

2.  Reclassify the Soil map into **Hydrological Soil Groups** (HSG: $A$ = high infiltration, $B$ = moderate, $C$ = slow, $D$ = very slow/clay).

3.  Load a lookup matrix table linking LULC classes and HSG letters to their respective CN values.

4.  Use **Union** or **Vector Overlay** to combine the LULC and Soil layers.

5.  Join the lookup table to append the correct CN attribute based on the combined land/soil code.

6.  Rasterize the polygon layer to generate a continuous $30\text{ m}$ CN grid. Run **Zonal Statistics** to compute the area-weighted average CN for each delineated sub-basin in the model.

---

## 5. Hydrological Significance

*   **Peak Flow Forecasting:** Calibrated HEC-HMS models running CN parameter grids predict flood peaks at bridge sites during extreme precipitation events.

*   **Land Use Impact Simulation:** By modifying the LULC raster in QGIS (e.g., converting $50\%$ of a forest catchment to urban space) and re-running the SWAT model, hydrologists can quantify how future urbanization increases peak flood discharge and reduces groundwater baseflow.

*   **Climate Change Adaptation:** Model parameters extracted via GIS help simulate catchment runoffs under future IPCC precipitation scenario projections, assisting in reservoir sizing audits.


## 6. Data Sources & Acquisition

For hydrological modeling inputs when local catchment databases are unavailable:

*   **Harmonized World Soil Database (HWSD):** Provides global soil textures, bulk densities, and hydrological soil groups (A, B, C, D) required to parameterize runoff Curve Numbers in SWAT and HEC-HMS. Download from the [IIASA HWSD Portal](https://gryphon.iiasa.ac.at/index.php/HWSD) or the [FAO Soil Portal](https://www.fao.org/soils-portal/data-hub/soil-maps-and-databases/).

*   **Global Land Cover maps:** Download LULC grids from the [ESA WorldCover Portal](https://esa-worldcover.org/) or [Copernicus Land Monitoring Service (CLMS)](https://land.copernicus.eu/global/).

*   **Meteorological Model Inputs (NASA POWER):** Download historical daily temperature, solar radiation, humidity, and wind speed statistics (required to drive continuous models like SWAT) for your coordinate centroid using the [NASA POWER API & Viewer](https://power.larc.nasa.gov/).
