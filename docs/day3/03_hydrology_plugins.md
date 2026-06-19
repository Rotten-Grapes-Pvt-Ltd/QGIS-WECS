# Hydrological Plugins and Integrated Tools

While QGIS provides robust core tools for general spatial operations, specialized hydrological analyses require dedicated plugins and external modeling integrations. These tools extend QGIS to support watershed delineation, river hydraulics, 2D flood routing, groundwater transport simulation, and borehole database management.

---

## Hydrology Plugin Priority Matrix

The table below outlines the recommended progression for mastering hydrological plugins in QGIS.

| Learning Priority | Tool / Plugin Name | Primary Application |
| :--- | :--- | :--- |
| **Must Learn** | WhiteboxTools | High-performance terrain analysis, flow routing, and stream extraction. |
| **Must Learn** | GRASS Hydrology Tools | Catchment delineation, landscape flow accumulation, and channel statistics. |
| **Must Learn** | SAGA Hydrology Tools | Topographic wetness index, LS-factor, and slope height calculations. |
| **Advanced** | QSWAT / QSWAT+ | Soil & Water Assessment Tool (SWAT) interface for basin-scale runoff. |
| **Advanced** | RiverGIS | Cross-section geometry preparation for 1D HEC-RAS hydraulic models. |
| **Advanced** | FLO-2D | Visual simulation setup for 2D flood routing and urban inundation. |
| **Specialized** | FREEWAT | Integrated groundwater flow modeling (MODFLOW) and management. |
| **Specialized** | Midvatten | Hydrogeological database tool for borehole and groundwater level records. |
| **Specialized** | Giswater | Urban drainage, wastewater, and water supply network management. |

---

## 1. Must Learn: Core Processing Integrations

These integrations come packaged or run directly inside the QGIS Processing Toolbox. They are essential for every hydrologist to perform fundamental terrain analysis.

### WhiteboxTools (WBT)

**Repository/Docs:** [https://www.whiteboxgeo.com/](https://www.whiteboxgeo.com/)

An advanced, high-performance geospatial analysis engine written in Rust. It integrates into QGIS via the `WhiteboxToolbox` plugin, offering over 500 tools.

*   **Key Capabilities:** 
    *   Sinks filling and breaching (e.g., *Fill Depressions*, *Breach Depressions*).
    *   Advanced flow routing (D8, D-Infinity, FD8) and flow accumulation.
    *   Topographic metrics (slope, aspect, profile curvature, wetness index).
    *   Vector stream network extraction and cleanup.

*   **Hydrological Application:** WhiteboxTools is highly optimized for processing large LiDAR DEMs, enabling fast and precise drainage network extraction in flat terrain.

*   **Access in QGIS:** 
    1.  Install the **WhiteboxToolbox** plugin from the QGIS Plugin Repository.
    2.  Download the compiled WhiteboxTools executable binary from the official website.
    3.  Configure the executable binary path in QGIS via **Settings** > **Options** > **Processing** > **Providers** > **WhiteboxTools**.

---

### GRASS Hydrology Tools

**Documentation:** [https://grass.osgeo.org/](https://grass.osgeo.org/)

GRASS GIS is fully integrated into the QGIS Processing Toolbox, providing powerful command-line modules for hydrological modeling.

*   **Key Capabilities:**
    *   `r.watershed`: Calculates flow accumulation, drainage direction, stream networks, and sub-basins using a multi-flow direction (MFD) or single-flow direction (SFD) algorithm.
    *   `r.water.outlet`: Generates watershed boundaries for a single point of interest (pour point).
    *   `r.stream.extract`: Extracts stream networks from a DEM with user-defined threshold and direction settings.

*   **Hydrological Application:** Preferred for processing massive catchment basins where memory safety and robust topology are required.

*   **Access in QGIS:** Open the **Processing Toolbox** (Ctrl+Alt+T) and expand the **GRASS** provider group to access the `r.*` raster tools.

---

### SAGA Hydrology Tools

**Documentation:** [https://saga-gis.sourceforge.io/](https://saga-gis.sourceforge.io/)

SAGA GIS is a dedicated raster processing platform that connects seamlessly with QGIS, offering the widest range of terrain parameter calculations.

*   **Key Capabilities:**
    *   *SAGA Wetness Index (SWI)*: A modified Topographic Wetness Index that accounts for soil moisture storage capacity.
    *   *Slope Length and Steepness (LS) Factor*: A key input parameter for Soil Loss models like RUSLE.
    *   *Channel Network and Drainage Basins*: Generates clean vector streams and nested catchments.

*   **Hydrological Application:** Standard industry tool for calculating terrain derivatives and preparing raster inputs for erosion and runoff models.

*   **Access in QGIS:** Install the **SAGA NextGen** plugin from the plugin manager to run SAGA tools directly from the Processing Toolbox.

---

## 2. Advanced: Numerical Model Interfaces

These plugins act as graphical user interfaces (GUIs) within QGIS, linking your spatial layers to external hydraulic and hydrological simulation engines.

### QSWAT and QSWAT+

**Documentation:** [https://swat.tamu.edu/software/qswatplus/](https://swat.tamu.edu/software/qswatplus/)

A Python-based interface for the Soil & Water Assessment Tool (SWAT/SWAT+), which is a river basin-scale, continuous-time model.

*   **Key Capabilities:**
    *   Delineates watersheds and divides them into Hydrologic Response Units (HRUs) based on soil, land use, and slope.
    *   Prepares climatological inputs (precipitation, temperature) and associates them with sub-basins.
    *   Runs the SWAT simulation engine and imports outputs (river discharge, sediment yield) back into QGIS for visualization.

*   **Hydrological Application:** Used for evaluating water resource management policies, agricultural runoff, and sediment transport in large catchments.

---

### RiverGIS

**Repository:** [https://github.com/folmer/RiverGIS](https://github.com/folmer/RiverGIS)

A tool designed to prepare geometric data for HEC-RAS, a 1D/2D hydraulic model developed by the US Army Corps of Engineers.

*   **Key Capabilities:**
    *   Generates cross-sectional lines across river centerlines.
    *   Extracts elevation profiles along cross-sections using a underlying high-resolution DEM.
    *   Establishes bank stations, flow paths, and downstream reach lengths.
    *   Exports spatial geometries directly to HEC-RAS geometry format (`.g01`).

*   **Hydrological Application:** Simplifies the tedious workflow of digitizing cross-sections and extracting elevations for 1D flood wave routing.

---

### FLO-2D

**Website:** [https://www.flo-2d.com/](https://www.flo-2d.com/)

An interface to configure the FLO-2D model, a physical-based flood routing model for channel flow, street routing, and overland flood plains.

*   **Key Capabilities:**
    *   Generates orthogonal computation grids from DEM raster inputs.
    *   Configures hydraulic boundary conditions (inflow hydrographs, outflow nodes).
    *   Assigns spatially variable roughness coefficients (Manning's n) using land-use shapefiles.
    *   Runs the FLO-2D simulation engine and visualizes velocities, depths, and hazards.

*   **Hydrological Application:** Standard model for urban flood hazard mapping, mudflow routing, and dam break simulations.

---

## 3. Specialized: Groundwater & Water Infrastructure

Specialized plugins designed for sub-surface water management, utility distribution, and environmental monitoring.

### FREEWAT

**Website:** [https://www.freewat.eu/](https://www.freewat.eu/)

FREEWAT (FREE and open source software tools for Water resource management) is an integrated environment for groundwater flow and solute transport modeling.

*   **Key Capabilities:**
    *   Couples QGIS vector layers with the **MODFLOW** simulation engine.
    *   Models groundwater recharge, well extractions, evapotranspiration, and aquifer boundaries.
    *   Supports solute transport models (MT3DMS) and seawater intrusion analysis.

*   **Hydrological Application:** Used for modeling aquifer drawdown, groundwater pollution plumes, and sustainable drinking water extraction rates.

---

### Midvatten

**Repository:** [https://github.com/jkall/qgis-midvatten](https://github.com/jkall/qgis-midvatten)

A plugin for managing and analyzing hydrogeological database records.

*   **Key Capabilities:**
    *   Connects QGIS to SQLite/SpatiaLite databases containing borehole profiles.
    *   Generates lithological cross-sections and borehole logs.
    *   Plots groundwater level timeseries, Piper diagrams, and Schoeller water quality charts.

*   **Hydrological Application:** Crucial for hydrogeological field investigations, drilling campaigns, and tracking seasonal aquifer table fluctuations.

---

### Giswater

**Website:** [https://www.giswater.org/](https://www.giswater.org/)

An open-source driver designed to connect spatial databases (PostgreSQL/PostGIS) to hydraulic simulation engines (EPA-SWMM for drainage and EPANET for water supply).

*   **Key Capabilities:**
    *   Converts complex water asset networks (pipes, valves, junctions, catchpits) into mathematical simulation models.
    *   Configures network topologies and runs EPANET or SWMM simulations.
    *   Saves simulation logs and results back to a multi-user PostGIS database.

*   **Hydrological Application:** Standard tool for public utility operators managing urban water distribution networks, combined sewer overflows (CSOs), and stormwater drainage.
