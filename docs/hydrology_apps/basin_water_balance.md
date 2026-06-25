# Basin Water Balance in QGIS

A basin water balance calculates the inputs, outputs, and storage changes of water within a defined catchment area. By utilizing gridded satellite precipitation and evapotranspiration data, water resource managers can map water availability, estimate groundwater recharge, and pinpoint zones of high water stress.

> [!TIP]
> **Data Sources & Acquisition:**
> To calculate a basin-scale water budget grid, download these free global raster products:
> 
> *   **Precipitation (P):** Download daily or monthly CHIRPS GeoTIFF grids from the [UCSB Climate Hazards Center CHIRPS Directory](https://www.chc.ucsb.edu/data/chirps) or GPM IMERG grids from the [NASA Earthdata Search](https://search.earthdata.nasa.gov/).
> 
> *   **Actual Evapotranspiration (ET):** Download actual evapotranspiration ($ET_a$) grids at 100m resolution from the [FAO WaPOR Portal](https://wapor.apps.fao.org/) or obtain global SSEBop actual ET grids from the [USGS FEWS NET Data Portal](https://earlywarning.usgs.gov/fews/product/66).
> 
> *   **Runoff Coefficients (C):** Local soil-cover runoff coefficient tables or curve numbers can be used to model runoff splits.

---

## 1. Core Objectives

*   **Align** multi-source satellite grids (Precipitation and Evapotranspiration) to a matching spatial system.

*   **Compute** grid-cell water surplus and deficit maps using raster algebra.

*   **Partition** surplus water into estimated surface runoff and groundwater infiltration.

*   **Tabulate** volumetric water balances per sub-catchment polygon.

---

## 2. Key GIS Inputs

*   **Precipitation Grid:** Projected rainfall raster (`precipitation_utm.tif`).

*   **Evapotranspiration Grid:** Projected actual evapotranspiration raster (`et_actual_utm.tif`).

*   **Sub-catchment Boundaries:** Vector polygon layer (`sub_basins.gpkg`).

---

## 3. Step-by-Step Basin Water Balance Workflow

Calculating a catchment water budget requires grid alignment, cell-by-cell raster subtraction, runoff/infiltration partitioning, and zonal polygon summarization:

![flow_chart](images/basin_water_balance/flow_chart.png)

1.  **Align Raster Grids (Precipitation and Evapotranspiration):**
    
    *   **What We Are Doing:** Matching the grid size, cell resolution, and spatial extent of the precipitation and evapotranspiration rasters.
    
    *   **Why This Step is Needed:** To perform cell-by-cell mathematical operations (subtracting $ET$ from $P$), both rasters must have identical dimensions and bounding coordinates. Running an alignment step prevents grid offsets and mismatched cell boundaries.
    
    *   *Input:* Projected precipitation and evapotranspiration rasters (e.g. CHIRPS and WaPOR).
    
    *   *Output:* Aligned rasters (`precip_aligned.tif` and `et_aligned.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to the main QGIS menu and select **Raster** > **Align Rasters...**.
        
        *   **Rasters to Align:** Click the **Add** (+) button to add your precipitation and evapotranspiration rasters.
        
        *   **Reference Raster:** Select the precipitation raster.
        
        *   **Cell Size:** Keep the reference cell size (e.g., `100` meters).
        
        *   **Resampling Method:** Select **Bilinear** for continuous climate values.
        
        *   **Output Paths:** Save the aligned outputs as `precip_aligned.tif` and `et_aligned.tif`, then click **OK**.

2.  **Calculate Net Water Surplus / Deficit Grid:**
    
    *   **What We Are Doing:** Subtracting the evapotranspiration raster from the precipitation raster on a cell-by-cell basis.
    
    *   **Why This Step is Needed:** The difference between incoming precipitation and outgoing evapotranspiration ($P - ET$) represents the net water surplus available for surface runoff and groundwater recharge. Cells with positive values indicate water surplus (generation zones), while negative values indicate water deficit (soil moisture depletion).
    
    *   *Input:* Aligned rasters (`precip_aligned.tif` and `et_aligned.tif`) from **Step 1**.
    
    *   *Output:* Net water surplus raster grid (`water_surplus.tif`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to main menu **Raster** > **Raster Calculator...**.
        
        *   **Expression:** Enter the subtraction formula:
            `"precip_aligned@1" - "et_aligned@1"`
        
        *   **Output Layer:** Save as `water_surplus.tif`.

3.  **Partition Surplus into Surface Runoff and Infiltration:**
    
    *   **What We Are Doing:** Multiplying the water surplus grid by a runoff coefficient grid to estimate surface runoff and groundwater recharge.
    
    *   **Why This Step is Needed:** Not all surplus water drains into rivers as surface runoff; a significant portion infiltrates into soil and aquifers. Using a spatial runoff coefficient ($C$) raster based on land use and soil slope, we can model this split:
        
        $$\text{Runoff} = C \times \text{Surplus}$$
        
        $$\text{Infiltration} = (1 - C) \times \text{Surplus}$$
        
    *   *Input:* Net water surplus grid (`water_surplus.tif`) and spatial runoff coefficient grid (`runoff_coeff.tif`).
        
        > [!NOTE]
        > **Deriving the Runoff Coefficient ($C$) Grid if Unknown:**
        > If local measurements of the runoff coefficient are unavailable, you can generate a spatial $C$ raster in QGIS by assigning standard Rational Method runoff coefficients to your **Land Use (LULC)** classes:
        > 
        > | LULC Class | Slope < 2% (Flat) | Slope 2%–7% (Moderate) | Slope > 7% (Steep) |
        > | :--- | :--- | :--- | :--- |
        > | **Trees / Forest (ESA Class 10)** | 0.05 - 0.10 | 0.10 - 0.15 | 0.15 - 0.25 |
        > | **Cropland (ESA Class 40)** | 0.10 - 0.15 | 0.15 - 0.20 | 0.20 - 0.40 |
        > | **Built-up / Urban (ESA Class 50)** | 0.70 - 0.75 | 0.75 - 0.85 | 0.85 - 0.95 |
        > 
        > *GIS Derivation Steps:*
        > 1. Run **SAGA** > **Grid - Tools** > **Reclassify Grid Values** on your projected LULC raster (e.g. `lulc_utm.tif`).
        > 2. Define simple ranges to map class codes to their respective runoff coefficient values (e.g. mapping urban code `50` to `0.80`, forest code `10` to `0.10`).
        > 3. Save the resulting decimal raster grid as `runoff_coeff.tif` to use in the step below.
    
    *   *Output:* Estimated runoff grid (`runoff_depth.tif`) and infiltration grid (`recharge_depth.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Calculus** > **Grid Calculator**.
        
        *   **Grids:** Select `water_surplus.tif` as Grid 1 (`g1`) and `runoff_coeff.tif` as Grid 2 (`g2`).
        
        *   **Formula:** Enter: `g1 * g2` (for Runoff).
        
        *   **Result:** Save output as `runoff_depth.tif`.
        
        *   *For Infiltration:* Run the calculator again with the formula `g1 * (1 - g2)` and save as `recharge_depth.tif`.

4.  **Extract Catchment Volumetric Statistics (SAGA Raster Statistics for Polygons):**
    
    *   **What We Are Doing:** Summing the pixel values of the water balance grids within each sub-catchment boundary to calculate total volumes in cubic meters.
    
    *   **Why This Step is Needed:** Catchment planning requires water budgets expressed as total volumetric values ($m^3$) per sub-basin. SAGA's polygon stats tool extracts the mean depth ($mm$) and cell count to compute these volumes.
    
    *   *Input:* Sub-catchment polygons (`sub_basins.gpkg`) and aligned rainfall raster (`precip_aligned.tif`).
    
    *   *Output:* Attribute table updated with mean rainfall depths and total volume statistics.
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Raster Statistics for Polygons**.
        
        *   **Grid:** Select `precip_aligned.tif`.
        
        *   **Polygons:** Select `sub_basins.gpkg`.
        
        *   **Method:** Check the **mean** and **sum** statistic boxes.
        
        *   **Result:** Save as a new layer `sub_basin_water_budget.gpkg`.
        
        *   *Note:* Repeat this step for `et_aligned.tif`, `runoff_depth.tif`, and `recharge_depth.tif` to compile all components of the water balance table.

5.  **Calculate Final Water Balance Table using Field Calculator:**
    
    *   **What We Are Doing:** Converting calculated spatial statistics into volumetric values and checking for mass balance closure in the attribute table.
    
    *   **Why This Step is Needed:** Tabulating all components inside the attribute table enables simple water budgeting reports, exporting data to charts, and identifying sub-catchments experiencing chronic water deficits.
    
    *   *Input:* Attribute table of `sub_basin_water_budget.gpkg`.
    
    *   *Output:* Populate columns for precipitation volume ($V_p$), evapotranspiration volume ($V_{et}$), and storage change ($\Delta S$).
    
    *   *How to Fill the Form in QGIS:*
        
        *   Open the attribute table of `sub_basin_water_budget.gpkg` and open **Field Calculator**.
        
        *   **Calculate Precipitation Volume ($m^3$):**
            
            *   *Field Name:* `Vol_Precip` (Decimal number).
            
            *   *Expression:* `("precip_mean" / 1000) * $area`
            
        *   **Calculate Evapotranspiration Volume ($m^3$):**
            
            *   *Field Name:* `Vol_ET` (Decimal number).
            
            *   *Expression:* `("et_mean" / 1000) * $area`
            
        *   **Calculate Net Available Water / Storage Change ($\Delta S$, $m^3$):**
            
            *   *Field Name:* `Net_Surplus` (Decimal number).
            
            *   *Expression:* `"Vol_Precip" - "Vol_ET" - "Vol_Runoff"`

---

## 4. Analytical Methods and Mass Balance Closure

Hydrologists use the mass conservation equation to close the water balance:

$$P - ET - Q - G - \Delta S = \epsilon$$

Where $P$ is precipitation, $ET$ is evapotranspiration, $Q$ is surface runoff, $G$ is deep groundwater outflow, $\Delta S$ is change in soil storage, and $\epsilon$ is the water balance closure error. 

In satellite-based calculations, $\epsilon$ is rarely zero due to sensor resolution limitations, grid resamplings, and coefficient uncertainties. If $\epsilon$ exceeds $10\%$ to $15\%$ of total precipitation, the model coefficients must be calibrated against measured streamflow gauging data at the outlet.

---

## 5. Hydrological & Environmental Significance

*   **Sustainable Yield Mapping:** Sub-catchments with consistently negative net surplus values ($\Delta S < 0$) are consuming more water than is naturally replenished, signaling unsustainable groundwater pumping or high forest cover depletion.

*   **Dry Season Flow Forecasting:** High recharge/infiltration values (`recharge_depth.tif`) indicate high groundwater baseflow potential, maintaining dry season streamflows.

*   **Watershed Management Auditing:** Provides a transparent grid-based framework to evaluate how land use changes (e.g. converting agricultural cropland to urban centers) impact local water budgets.
