# Water Accounting (WA+ Style)

Water Accounting Plus (WA+) is a standard framework developed by IHE Delft, FAO, and partners. It uses open-access satellite data to report on the state of water resources, consumption, and flows at the river basin level. It acts as a spatial financial audit for water, showing where water comes from, who consumes it, and where it goes.

---

## 1. Core Objectives

*   **Audit** basin-scale water resources availability using spatial grids.

*   **Quantify** water consumption (evapotranspiration) across different land-use sectors.

*   **Evaluate** the basin's water balance closure and identify outflow surpluses or deficits.

---

## 2. Key GIS Inputs

*   **Precipitation ($P$):** Spatial rainfall grids (such as CHIRPS or IMERG).

*   **Actual Evapotranspiration ($ET_a$):** Spatial actual evapotranspiration and interception layers (such as FAO's WaPOR, SSEBop, or MOD16).

*   **Storage Change ($\Delta S$):** Changes in soil moisture, snow cover, and groundwater (from GRACE satellite anomalies or model ensembles).

*   **Land Use / Land Cover (LULC):** Standard classification map categorized into WA+ classes (Protected Areas, Utilized Land, Modified Land, Municipal/Industrial).

---

## 3. The Water Balance Equation

In a closed catchment, the water balance is calculated over a specific time period:

$$P + Q_{\text{in}} - ET_a - \Delta S - Q_{\text{out}} = \epsilon$$

Where:

*   $P$ = Total precipitation volume ($\text{m}^3$).

*   $Q_{\text{in}}$ = Transboundary surface/sub-surface water inflow ($\text{m}^3$).

*   $ET_a$ = Actual Evapotranspiration volume ($\text{m}^3$).

*   $\Delta S$ = Total storage change (groundwater + soil moisture + surface water, $\text{m}^3$).

*   $Q_{\text{out}}$ = River discharge outflow exiting the basin ($\text{m}^3$).

*   $\epsilon$ = Water balance closure error (due to satellite measurement noise).

*   *GIS Toolpaths (Grid Math):*
    
    *   **SAGA GIS:** **Processing Toolbox** > **SAGA** > **Grid - Calculus** > **Grid Calculator**.
    
    *   **WhiteboxTools:** **Processing Toolbox** > **WhiteboxTools** > **Math and Stats Tools** > **RasterCalculator**.

---

## 4. The Evapotranspiration Split ($ET_a$)

Under the WA+ framework, Actual Evapotranspiration is split into three primary physical fluxes:

$$ET_a = T + E + I$$

Where:

*   **Transpiration ($T$):** Water absorbed by plant roots, used for growth, and transpired through leaf stomata. This is productive water consumption.

*   **Evaporation ($E$):** Water evaporated directly from bare soils, wet canopies, and open water bodies. This is unproductive water consumption.

*   **Interception ($I$):** Rainfall caught by vegetation canopies that evaporates directly back to the atmosphere before reaching the ground.

*   *GIS Calculation:* WaPOR provides separate $T$ and $E$ grids. Using zonal statistics, water managers can calculate the **agricultural water productivity**:
    
    $$\text{Crop Water Productivity} = \frac{\text{Harvested Yield (kg)}}{\text{Transpiration } T \text{ (m}^3)}$$

*   *GIS Toolpaths (Zonal Statistics):*
    
    *   **SAGA GIS:** **Processing Toolbox** > **SAGA** > **Grid - Tools** > **Raster Statistics for Polygons**.
    
    *   **WhiteboxTools:** **Processing Toolbox** > **WhiteboxTools** > **Math and Stats Tools** > **ZonalStatistics**.

---

## 5. WA+ Land Use Classification Sheets

WA+ groups spatial land use polygons into four management classes:

1.  **Protected Land:** Protected national parks, high-altitude conservation zones, and wetlands. Little to no human water management.

2.  **Utilized Land:** Low-intensity grazing lands, natural forests, and rainfed agriculture. Natural ecosystems modified by human activity.

3.  **Modified Land:** Irrigated agriculture, plantation forestry, and livestock schemes. Heavy human water diversion and consumption.

4.  **Municipal / Industrial:** Urban centers, industrial nodes, and mining zones. High return-flow potential but high pollution hazards.

---

## 6. Hydrological Significance

*   **Transboundary Water Governance:** Provides a neutral, satellite-verified audit of water flows between countries, reducing political disputes over shared river systems.

*   **Irrigation Audit:** Identifies irrigation schemes consuming more water than allocated, assisting in upgrading canal operations.

*   **Sustainable Aquifer Yields:** Highlights basins where groundwater extraction exceeds natural recharge rates by tracking chronic negative storage anomalies ($\Delta S$).


## 7. Data Sources & Acquisition

If you do not have water budget grids or evapotranspiration maps:

*   **Actual Evapotranspiration ($ET_a$):** Download evapotranspiration, transpiration, and interception raster datasets directly from the [FAO WaPOR Portal](https://wapor.apps.fao.org/) (spatial resolutions range from $100	ext{m}$ to $250	ext{m}$). Alternatively, download SSEBop actual ET grids from the [USGS FEWS NET Data Portal](https://earlywarning.usgs.gov/fews/product/66).

*   **Precipitation Grids ($P$):** Download CHIRPS or IMERG datasets.

*   **Groundwater Storage Anomalies ($\Delta S$):** Download GRACE and GRACE-FO spherical harmonic grids from the [NASA JPL GRACE Tellus Portal](https://grace.jpl.nasa.gov/) to map regional groundwater depletion and recharge storage cycles.
