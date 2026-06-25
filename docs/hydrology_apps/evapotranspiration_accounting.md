# Evapotranspiration Accounting

Evapotranspiration (ET) represents the combined process of water evaporated from soil and water transpired by vegetation canopies. Accounting for ET across different watersheds and land use classes allows water resource managers to determine a basin's water yield efficiency, identify "thirsty" vs. "water-supplying" catchments, and evaluate the environmental impact of vegetation cover changes.

> [!TIP]
> **Data Sources & Acquisition:**
> To perform spatial evapotranspiration accounting, obtain the following free global satellite raster datasets:
> 
> *   **Actual Evapotranspiration (ETa):** Download monthly ETa rasters (100m resolution) from the [FAO WaPOR Portal](https://wapor.apps.fao.org/) or download monthly SSEBop actual ET grids (1km resolution) from the [USGS FEWS NET Data Portal](https://earlywarning.usgs.gov/fews/product/66).
> 
> *   **Precipitation (P):** Download matching monthly CHIRPS GeoTIFF grids from the [UCSB Climate Hazards Center CHIRPS Directory](https://www.chc.ucsb.edu/data/chirps).
> 
> *   **Land Use / Land Cover (LULC):** Download the ESA WorldCover 10m land cover grid from the [ESA WorldCover Portal](https://esa-worldcover.org/).

---

## 1. Core Objectives

*   **Compute** zonal actual evapotranspiration volumes across multiple watershed basins.

*   **Extract** land cover-specific ET consumption rates using spatial raster masking.

*   **Calculate** the Evapotranspiration Fraction (ET Ratio) to classify water yield efficiency.

*   **Compare** multiple sub-catchments to identify which watersheds act as net water suppliers vs. water consumers.

---

## 2. Key GIS Inputs

*   **Evapotranspiration Grid:** Projected actual ET raster (`et_actual_utm.tif`).

*   **Precipitation Grid:** Projected rainfall raster (`precipitation_utm.tif`).

*   **LULC Grid:** Projected land cover classification raster (`lulc_utm.tif`).

*   **Sub-catchments Layer:** Vector polygon layer (`sub_basins.gpkg`).

---

## 3. Step-by-Step Evapotranspiration Accounting Workflow

Spatial ET accounting requires grid alignment, zonal statistics for bulk catchment water consumption, raster-masking for land cover-specific ET rates, and table-based basin performance comparison:

```text
       [ Evapotranspiration Grid ]  +  [ LULC Classification Grid ]
                    │                             │
                    ├─────────────────────────────┤ (Raster Calculator Masking)
                    │                             ▼
                    │                   [ Forest-Specific ET Grid ]
                    │                             │
                    ▼                             ▼
       (SAGA Raster Statistics for Polygons using Sub-catchment boundaries)
                    │                             │
                    ▼                             ▼
        [ Bulk Catchment ET (Vol) ]     [ Forest ET Volume per Basin ]
                    │                             │
                    └───────────────┬─────────────┘
                                    ▼ (QGIS Field Calculator)
                     [ ET/P Ratios and LULC ET Tables ]
                                    │
                                    ▼ (Environmental Audit)
                     [ Multi-Basin Performance Analysis ]
```

1.  **Extract Bulk Catchment Evapotranspiration Volumes:**
    
    *   **What We Are Doing:** Summing the actual ET pixel values within the boundaries of each sub-catchment to compute total volumetric ET water loss ($m^3$).
    
    *   **Why This Step is Needed:** Evapotranspiration represents water returning to the atmosphere, meaning it is a net loss to the basin's available surface water. Calculating this volume allows us to audit how much water is lost to the atmosphere compared to what enters via rainfall.
    
    *   *Input:* Sub-catchment boundaries (`sub_basins.gpkg`) and aligned actual ET raster (`et_actual_utm.tif`).
    
    *   *Output:* Attribute table updated with mean ET depth ($mm$) and total volume statistics.
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Raster Statistics for Polygons**.
        
        *   **Grid:** Select `et_actual_utm.tif`.
        
        *   **Polygons:** Select `sub_basins.gpkg`.
        
        *   **Method:** Check the **mean** and **sum** statistic boxes.
        
        *   **Result:** Save as a new layer `sub_basin_et_budget.gpkg`.
        
        *   *Calculate Volume:* Open the attribute table of `sub_basin_et_budget.gpkg`, open **Field Calculator**, create a new decimal field named `Vol_ET_m3`, and enter the conversion equation:
            `("et_mean" / 1000) * $area`

2.  **Mask Evapotranspiration by LULC Class (Raster Calculator):**
    
    *   **What We Are Doing:** Creating a masked ET raster that only contains ET values where the land cover matches a specific class (e.g. Forest, Cropland), setting all other cells to 0 (no data).
    
    *   **Why This Step is Needed:** To find out how much water different land uses consume, we must isolate the ET pixels belonging to specific classes. This prevents mixed-pixel signature contamination.
    
    *   *Input:* Aligned actual ET raster (`et_actual_utm.tif`) and LULC raster (`lulc_utm.tif`).
    
    *   *Output:* Forest-specific ET raster (`et_forest.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the conditional masking formula for Forest (ESA class 10):
            `"et_actual_utm@1" * ("lulc_utm@1" == 10)`
            *(This formula multiplies the ET value by 1 where the LULC code equals 10, and by 0 everywhere else, effectively isolating forest ET).*
        
        *   **Output Layer:** Save as `et_forest.tif`.
        
        *   *For Cropland:* Run again with `"et_actual_utm@1" * ("lulc_utm@1" == 40)` and save as `et_cropland.tif`.

3.  **Extract LULC-Specific ET Volumes per Catchment:**
    
    *   **What We Are Doing:** Summing the masked land cover ET pixels within each sub-catchment polygon.
    
    *   **Why This Step is Needed:** This allows us to quantify the exact volume of water consumed specifically by forests or irrigated agricultural crops inside each individual sub-catchment.
    
    *   *Input:* Sub-catchment boundaries (`sub_basin_et_budget.gpkg`) and masked ET grids (`et_forest.tif`, `et_cropland.tif`).
    
    *   *Output:* Columns added to the attribute table (e.g. `Vol_ET_Forest`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Raster Statistics for Polygons**.
        
        *   **Grid:** Select `et_forest.tif`.
        
        *   **Polygons:** Select `sub_basin_et_budget.gpkg`.
        
        *   **Method:** Check the **mean** and **sum** statistic boxes.
        
        *   **Result:** Save the output as a new layer. In the attribute table, run the Field Calculator to create a new decimal field named `Vol_ET_Forest` using:
            `("forest_mean" / 1000) * $area`

4.  **Compute Catchment Water Yield and Evapotranspiration Ratios:**
    
    *   **What We Are Doing:** Calculating the ET-to-Precipitation ratio ($ET_{\text{ratio}}$) to evaluate the hydrological water yield efficiency of each basin.
    
    *   **Why This Step is Needed:** The ratio of actual evapotranspiration to precipitation represents the catchment's water consumption index. Catchments with high ET ratios act as water sinks, while catchments with low ET ratios yield water downstream.
    
    *   *Input:* Attribute table of `sub_basin_et_budget.gpkg` containing bulk precipitation volume (`Vol_Precip` derived from Rainfall Zonal Stats) and bulk ET volume (`Vol_ET_m3`).
    
    *   *Output:* Attribute field `ET_Ratio`.
    
    *   *How to Fill the Form in QGIS:*
        
        *   Open the attribute table and open **Field Calculator**.
        
        *   Create a new decimal field named `ET_Ratio`.
        
        *   **Expression:** Enter the ratio formula:
            `"Vol_ET_m3" / "Vol_Precip"`

---

## 4. Multi-Basin Performance Comparison Matrix

Once the statistics are extracted, compile a comparison matrix to evaluate and rank basin environmental performance:

| Sub-basin | Precipitation (P, m³) | Evapotranspiration (ET, m³) | $ET_{\text{ratio}}$ | Forest ET (m³) | Cropland ET (m³) | Hydrological Classification |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Basin A (High Forest)** | 10,000,000 | 4,000,000 | **0.40** | 3,500,000 | 200,000 | **Water Supplier / Recharge Zone** |
| **Basin B (Irrigated Crop)**| 8,000,000 | 7,200,000 | **0.90** | 500,000 | 6,500,000 | **Water Consumer / High Stress** |
| **Basin C (Urbanized)** | 9,500,000 | 2,850,000 | **0.30** | 100,000 | 100,000 | **Flashy Runoff / Infiltration Depleted**|

### How to Evaluate Which Watershed is "Better":
*   **The Best Water Supplier (Basin A):** Has a low $ET_{\text{ratio}}$ (0.40), meaning **60%** of its precipitation is conserved to generate streamflow runoff and groundwater recharge. It acts as the primary water source for downstream populations.
*   **The Thirsty Catchment (Basin B):** Has a very high $ET_{\text{ratio}}$ (0.90), consuming **90%** of its precipitation through crop evapotranspiration. It yields very little water downstream and is vulnerable to drought water deficits.
*   **The Distorted Catchment (Basin C):** Shows the lowest $ET_{\text{ratio}}$ (0.30), but this is due to concrete imperviousness (low forest/crop transpiration). It generates high flash-flood runoff volumes with zero groundwater recharge.

---

## 5. Hydrological & Environmental Significance

*   **Forest Hydrology Management:** Verifies if headwater forests are healthy and transpirative. High forest ET indicates active vegetation growth and healthy catchment soil retention.

*   **Irrigation Audit & Water Productivity:** Estimates if irrigated fields are consuming excessive groundwater. Managers can enforce crop changes if crop ET exceeds sustainable basin limits.

*   **Land Use Policy Feedback:** Guides municipal planning by showing where forest degradation or urbanization will reduce available recharge and downstream reservoir refills.
