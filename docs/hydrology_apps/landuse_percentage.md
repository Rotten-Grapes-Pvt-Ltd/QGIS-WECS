# Land Use Area Calculation per Sub-catchment

Land Use and Land Cover (LULC) determines the partition of precipitation into infiltration, evapotranspiration, and surface runoff. By intersecting high-resolution global land cover datasets (such as the ESA WorldCover 10m grid) with delineated sub-catchment boundaries, hydrologists can calculate the exact percentage of land use classes per basin to evaluate environmental impacts and configure watershed models.

> [!TIP]
> **Data Sources & Acquisition:**
> For local or global land use assessments, download the following free high-resolution land cover datasets:
> 
> *   **ESA WorldCover (10m resolution):** Global land cover map based on Sentinel-1 and Sentinel-2 data. Contains 11 land cover classes. Download tiles in GeoTIFF format directly from the [ESA WorldCover Portal](https://esa-worldcover.org/).
> 
> *   **Esri Land Cover (10m resolution):** Global 10-class land use/land cover map derived from Sentinel-2. Available via the [Esri Land Cover Explorer](https://livingatlas.arcgis.com/landcover/).
> 
> *   **Copernicus Global Land Cover (100m resolution):** Annual global land cover maps from the Copernicus Land Monitoring Service (CLMS). Download from the [Copernicus Land Portal](https://land.copernicus.eu/global/products/lc).

---

## 1. Core Objectives

*   **Import** and align the ESA WorldCover raster with sub-catchment vector boundaries.

*   **Reproject** LULC datasets to a metric coordinate system to preserve geometry for area calculations.

*   **Extract** pixel counts of individual land cover classes within each sub-catchment boundary.

*   **Calculate** the percentage area of each land cover type and evaluate the corresponding environmental and hydrological impacts.

---

## 2. Key GIS Inputs

*   **LULC Classification Grid:** ESA WorldCover GeoTIFF (`ESA_WorldCover_10m.tif`).

*   **Sub-catchments Layer:** Delineated sub-basin vector polygon layer (`sub_basins.gpkg`).

---

## 3. Step-by-Step Land Use Calculation Workflow

Calculating land use percentages per sub-catchment requires reprojecting categorical grids, extracting class histograms, and computing area percentages:

![flow_chart](images/landuse_percentage/flow_chart.png)

1.  **Reproject LULC Grid to Match Sub-catchments projected CRS:**
    
    *   **What We Are Doing:** Transforming the coordinate reference system (CRS) of the LULC raster from degrees (geographic, e.g., WGS 84 / `EPSG:4326`) to meters (projected metric coordinate system, e.g., UTM Zone 45N / `EPSG:32645`).
    
    *   **Why This Step is Needed:** ESA WorldCover is distributed in geographic degrees. Measuring areas in degrees yields highly distorted values. Reprojecting to a metric system (like UTM) is mandatory so that cell count calculations can be converted directly into accurate square meters and hectares.
    
    *   *Input:* Geographic coordinate LULC raster (`ESA_WorldCover_10m.tif`).
    
    *   *Output:* Projected metric LULC raster (`lulc_utm.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to the main QGIS menu and select **Raster** > **Projections** > **Warp (Reproject)...**.
        
        *   **Input Layer:** Select `ESA_WorldCover_10m.tif`.
        
        *   **Target CRS:** Select your local UTM projected system (e.g., `EPSG:32645` for UTM Zone 45N).
        
        *   **Resampling Method:** Select **Nearest Neighbor** (this is critical for discrete categorical rasters like LULC; interpolations like Bilinear would calculate invalid decimal classes between forest and water).
        
        *   **Converted:** Save the output as `lulc_utm.tif`.

2.  **Extract LULC Class Counts per Sub-catchment (Zonal Histogram):**
    
    *   **What We Are Doing:** Overlaying the sub-catchment polygons on the LULC raster to count the number of pixels of each distinct land cover class inside each sub-catchment.
    
    *   **Why This Step is Needed:** To find the area of a class, we must first find how many raster grid cells of that class fall within the boundary of each sub-catchment. QGIS's Zonal Histogram tool counts these cells and automatically appends them as new fields in the vector layer's attribute table.
    
    *   *Input:* Sub-catchment polygons (`sub_basins.gpkg`) and projected LULC raster (`lulc_utm.tif`) from **Step 1**.
    
    *   *Output:* Vector polygon layer (`sub_basins_histogram.gpkg`) populated with `HISTO_<ClassValue>` columns.
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **Raster Analysis** > **Zonal Histogram**.
        
        *   **Raster Layer:** Select `lulc_utm.tif`.
        
        *   **Vector Layer Containing Zones:** Select `sub_basins.gpkg`.
        
        *   **Output Prefix:** Type `LULC_`.
        
        *   **Output Layer:** Save the output as a GeoPackage named `sub_basins_histogram.gpkg`.
        
        *   **Run the Tool:** The tool will generate a copy of the polygon layer, adding columns like `LULC_10` (pixel count of class 10), `LULC_40` (pixel count of class 40), etc.

3.  **Compute Percentage of Land Use Classes per Sub-catchment:**
    
    *   **What We Are Doing:** Converting raw pixel counts into percentage fractions of the total sub-catchment area using QGIS Field Calculator.
    
    *   **Why This Step is Needed:** Pixel counts must be normalized into percentage values to compare different sub-catchments and input them into environmental index calculations. For a 10m grid resolution, each cell covers $10\text{m} \times 10\text{m} = 100\text{ m}^2$.
    
    *   *Input:* Vector polygon layer (`sub_basins_histogram.gpkg`) from **Step 2**.
    
    *   *Output:* Sub-catchment layer containing new columns for percentage area (e.g., `PCT_Forest`, `PCT_Urban`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   Right-click `sub_basins_histogram.gpkg` and select **Open Attribute Table**.
        
        *   Open the **Field Calculator** (Ctrl+I).
        
        *   **Create a new field:**
            
            *   *Output field name:* `PCT_Urban` (for Built-up, ESA class 50).
            
            *   *Output field type:* **Decimal number (real)**.
            
            *   *Expression:* Enter the following formula:
                `("LULC_50" * 100) / $area * 100`
                *(Note: `"LULC_50" * 100` calculates the area of the urban pixels in square meters, which is then divided by the total polygon `$area` in square meters and multiplied by 100 to get a percentage).*
        
        *   Click **OK**.
        
        *   Repeat the step for other classes (e.g., Forest `LULC_10` using `("LULC_10" * 100) / $area * 100`).

---

## 4. ESA WorldCover Class Reference Guide

Below is the standard land cover classification legend for the ESA WorldCover dataset:

| Value | LULC Class | Description |
| :--- | :--- | :--- |
| **10** | **Trees** | Forest cover, evergreen and deciduous trees |
| **20** | **Shrubland** | Woody perennial plants under 5m in height |
| **30** | **Grassland** | Herbaceous vegetation and pastures |
| **40** | **Cropland** | Cultivated agricultural fields, annual crops |
| **50** | **Built-up** | Urban cities, infrastructure, roads, and buildings |
| **60** | **Barren / Sparse** | Rocks, sand, soil, and extremely sparse vegetation |
| **70** | **Snow and Ice** | Glaciers and permanent snowpack |
| **80** | **Open Water** | Rivers, lakes, reservoirs, and oceans |
| **90** | **Wetland** | Herbaceous wetlands, swamps, marshes |
| **95** | **Mangroves** | Coastal mangrove forests |
| **100** | **Moss / Lichen** | Tundra vegetation, mosses, and lichens |

---

## 5. Environmental & Hydrological Impact Analysis

Calculating land use percentages allows hydrologists to predict catchment responses and environmental degradation:

*   **Urbanization and Peak Runoff (Built-up %):** 
    
    As the built-up area (`PCT_Urban`) exceeds **10% to 15%** of a sub-catchment, the natural soil surface is replaced by impervious concrete. This reduces soil infiltration, causing:
    
    *   Shorter Time of Concentration ($T_c$) for runoff routing.
    
    *   Flashy hydrographs with massive peak flood discharges.
    
    *   Reduced groundwater recharge (baseflow depletion), causing dry-season water scarcity.

*   **Agricultural Runoff (Cropland %):**
    
    High agricultural land use (`PCT_Cropland`) indicates high potential for non-point source pollution. Runoff carries fertilizers (nitrogen, phosphorus) and pesticides into stream networks, leading to:
    
    *   Eutrophication and algal blooms in downstream lakes/reservoirs.
    
    *   Sediment loading from tilled agricultural fields lacking soil-anchoring root systems.

*   **Forest Conservation Mitigation (Trees %):**
    
    High tree cover (`PCT_Forest`) acts as a natural buffer. Forest canopies intercept rainfall, and root networks increase soil macropores. This stabilizes catchments by:
    
    *   Increasing infiltration and recharging aquifers.
    
    *   Reducing soil erosion and reservoir sedimentation rates.
    
    *   Lowering peak flood volumes by smoothing discharge hydrographs.
