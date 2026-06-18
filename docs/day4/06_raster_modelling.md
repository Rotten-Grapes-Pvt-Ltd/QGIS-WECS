# Raster-Based Hydrological and Soil Erosion Modelling

Raster grids allow us to model complex spatial environmental processes (such as surface runoff generation and soil erosion rates) by combining multiple spatial datasets. By performing cell-by-cell calculations, the raster layout models how physical variables interact across the entire catchment basin. This section covers peak discharge modeling (the Rational Method), runoff depth modeling (the SCS Curve Number method), and soil erosion susceptibility mapping using the Revised Universal Soil Loss Equation (RUSLE).

---

## 1. Surface Runoff Peak Discharge Modeling (The Rational Method)

Estimating peak discharge is critical for flood hazard zone mapping, storm drain dimensioning, and culvert capacity calculations.

### Theory and Mathematical Formulation

The Rational Method is a simple empirical model used to calculate the peak surface runoff rate ($Q$) at a catchment outlet:

$$Q = C \times I \times A$$

To calculate peak discharge in metric units ($Q$ in $m^3/\text{sec}$), we apply a conversion factor based on the units of input parameters:

$$Q = \frac{C \times I \times A}{3.6 \times 10^6}$$

Where:

*   $Q$ is the peak runoff rate (in cubic meters per second, $m^3/\text{sec}$).

*   $C$ is the dimensionless runoff coefficient, representing the fraction of rainfall that becomes surface runoff.
    
    *   **Paved / Asphalt Areas:** $C = 0.90$ (high runoff potential)
    
    *   **Bare Clay Soils:** $C = 0.60$
    
    *   **Dense Forests:** $C = 0.15$ (low runoff, high interception/infiltration)

*   $I$ is the rainfall intensity (in millimeters per hour, $mm/\text{hr}$) corresponding to a storm duration equal to the basin's Time of Concentration ($t_c$).

*   $A$ is the catchment drainage area (in square meters, $m^2$) calculated from the flow accumulation boundary.

### Time of Concentration ($t_c$)

The time required for water to travel from the hydraulically most remote point in the watershed to the outlet, calculated using the Kirpich equation:

$$t_c = 0.0195 \times L^{0.77} \times S^{-0.385}$$

Where:

*   $t_c$ is the time of concentration in minutes.

*   $L$ is the maximum length of flow travel in meters.

*   $S$ is the average slope gradient of the channel ($H_{\text{drop}} / L$).

### Use and Interpretation

Peak runoff calculations define design limits for infrastructure. For example, a planned culvert at a road crossing must accommodate the peak 100-year return period runoff rate ($Q_{100}$) computed from the upstream drainage area.

### QGIS Analysis Steps

1.  **Delineate Catchment Area ($A$):**
    
    Ensure the vector catchment boundary (e.g. `sub_basins_polygons.gpkg` created in Chapter 5) is loaded. Open the attribute table and use the field calculator to compute area:
    
    `$area` (units in square meters).

2.  **Calculate Peak Runoff rate ($Q$) in QGIS:**
    
    *   Open **Raster** > **Raster Calculator...**.
    
    *   If you have a spatial runoff coefficient raster `runoff_coeff_C.tif` and a rainfall intensity grid `rain_intensity_I.tif`, input the metric Rational formula:
        
        `("runoff_coeff_C@1" * "rain_intensity_I@1" * [Catchment Area value]) / 3600000`
    
    *   Save the output raster as `peak_runoff_Q.tif`.
    
    *   Click **OK**.
    
    *   *Interpretation:* High values indicate local sub-basins generating severe volumes of peak flow, pointing to downstream flooding zones.

---

## 2. Runoff Volume Modeling (The SCS Curve Number Method)

Developed by the USDA Soil Conservation Service, this model calculates cumulative runoff depth based on land use, soil hydrologic properties, and antecedent soil moisture.

### Theory and Mathematical Formulation

*   **The Runoff Equation:**
    
    $$Q_d = \frac{(P - I_a)^2}{P - I_a + S} \quad \text{for } P > I_a$$
    
    Where:
    
    *   $Q_d$ is the cumulative runoff depth (in millimeters, $mm$).
    
    *   $P$ is the cumulative rainfall depth (in millimeters, $mm$).
    
    *   $I_a$ is the initial abstraction (water intercepted by vegetation, micro-depressions, and initial soil wetting, typically assumed to be $0.2 \times S$).
    
    *   $S$ is the maximum potential soil water retention capacity ($mm$) after runoff begins:
        
        $$S = \frac{25400}{CN} - 254$$

*   **Hydrologic Soil Groups (HSG):**
    
    Soils are categorized into four groups based on their saturated hydraulic conductivity ($K_{\text{sat}}$) and infiltration rates:
    
    *   **Group A (Sand, loamy sand):** High infiltration, low runoff potential ($K_{\text{sat}} > 40\text{ mm/hr}$).
    
    *   **Group B (Sandy loam, loam):** Moderate infiltration ($K_{\text{sat}} \in [10, 40]\text{ mm/hr}$).
    
    *   **Group C (Clayey loam, silty loam):** Low infiltration ($K_{\text{sat}} \in [1, 10]\text{ mm/hr}$).
    
    *   **Group D (Heavy clay, shallow soil over rock):** Very low infiltration, high runoff potential ($K_{\text{sat}} < 1\text{ mm/hr}$).

*   **SCS Curve Number (CN) Matrix Table (AMC II - Average Moisture):**
    
    | Land Cover Class | HSG A | HSG B | HSG C | HSG D |
    | :--- | :--- | :--- | :--- | :--- |
    | **Dense Forest** | 30 | 55 | 70 | 77 |
    | **Grassland / Pasture** | 39 | 61 | 74 | 80 |
    | **Agricultural Cultivation** | 67 | 78 | 85 | 89 |
    | **Urban Residential / Concrete** | 77 | 85 | 90 | 92 |

### Use and Interpretation

Outputs the spatial distribution of runoff depth across a catchment. Higher values indicate impervious land or heavy clay soils where rainfall cannot infiltrate and immediately converts to surface water volume.

### QGIS Analysis Steps

1.  **Prepare the Curve Number (CN) Grid:**
    
    *   Overlay LULC and HSG vector polygon layers in QGIS using **Vector** > **Geoprocessing Tools** > **Union**.
    
    *   Open the attribute table. Add a new field `CN_val` (integer) and use field calculator conditional expressions (or a join table) to assign Curve Numbers based on the matrix table (e.g. `CASE WHEN "LULC" = 'Forest' AND "HSG" = 'A' THEN 30 ... END`).
    
    *   Rasterize this vector layer: Go to **Raster** > **Conversion** > **Rasterize (Vector to Raster)...**. Set **Input Layer** to the unioned layer, **Field to use for burn-in value** to `CN_val`, output resolution matching the DEM, and save as `curve_number.tif`.

2.  **Calculate Potential Soil Water Retention ($S$):**
    
    *   Open **Raster** > **Raster Calculator...**.
    
    *   Enter the formula:
        
        `(25400 / "curve_number@1") - 254`
    
    *   Save output as `soil_retention_S.tif`.

3.  **Calculate Runoff Depth ($Q_d$):**
    
    *   Open the **Raster Calculator** again.
    
    *   Input the runoff equation (assuming a design storm rainfall $P = 150\text{ mm}$):
        
        `(("precipitation_P@1" - (0.2 * "soil_retention_S@1")) ^ 2) / ("precipitation_P@1" + (0.8 * "soil_retention_S@1"))`
    
    *   Save output as `runoff_depth_Qd.tif`.
    
    *   *Interpretation:* Style the output using a blue color ramp. Saturated soils and concrete surfaces will show peak runoff depths close to the rainfall value ($150\text{ mm}$), whereas forested sandy zones will show low values.

---

## 3. Soil Erosion Susceptibility (The RUSLE Model)

The **Revised Universal Soil Loss Equation (RUSLE)** models average annual soil loss caused by sheet and rill erosion:

$$A = R \times K \times LS \times C \times P$$

Where:

*   $A$ is the estimated average annual soil loss (in metric tons per hectare per year, $\text{t/ha/yr}$).

*   **Rainfall Erosivity Factor ($R$):**
    
    Measures the kinetic energy and intensity of falling raindrops.
    
    Derived by interpolating monthly or annual precipitation records ($P$, in $mm$) using empirical regional equations (e.g. Roose's equation: $R = 0.5 \times P \times 1.72$).

*   **Soil Erodibility Factor ($K$):**
    
    Reflects a soil's susceptibility to particle detachment and transport.
    
    Calculated based on clay, silt, sand, and organic carbon fractions. Standard values range from $0.01$ (low erodibility sandy soils) to $0.60$ (highly erodible silty soils).

*   **Topographic Factor ($LS$):**
    
    Combines slope length ($L$) and slope steepness ($S$).
    
    In complex terrain, this is calculated using the Moore & Burch (1986) specific catchment area formulation:
    
    $$LS = \left( \frac{\alpha}{22.13} \right)^m \times \left( \frac{\sin \theta}{0.0896} \right)^n$$
    
    Where $\alpha$ is the Specific Catchment Area (SCA) from flow accumulation, $\theta$ is the local slope angle in degrees, and $m$ and $n$ are empirical exponents ($m \approx 0.4 \text{ to } 0.6$, $n \approx 1.2 \text{ to } 1.3$).

*   **Cover Management Factor ($C$):**
    
    Reflects the vegetative canopy's ability to shield soil from raindrop impact.
    
    Ranges from $0.001$ (dense forest) to $1.0$ (barren soil). Often mapped using satellite-derived NDVI grids:
    
    $$C = \exp \left( -a \times \frac{\text{NDVI}}{\beta - \text{NDVI}} \right)$$

*   **Conservation Support Practice Factor ($P$):**
    
    Reflects the impact of soil conservation practices, such as terracing or contour farming.
    
    In steep Himalayan catchments, active terracing drops the $P$ value from $1.0$ (no support) to $0.1$, significantly reducing modeled soil loss.

### Use and Interpretation

Identifies spatial soil loss hotspots. Crucial for watershed rehabilitation planning (e.g., targeting soil bioengineering or agroforestry to zones showing severe erosion rates).

### QGIS Analysis Steps

1.  **Calculate the Topographic LS Factor in SAGA:**
    
    *   Ensure the conditioned DEM `filled_dem_wang_liu.tif` and the flow accumulation grid `flow_accumulation_unweighted.tif` (from Chapter 4) are loaded.
    
    *   Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **LS Factor**.
    
    *   **Elevation:** Select `filled_dem_wang_liu.tif`.
    
    *   **Flow Accumulation:** Select `flow_accumulation_unweighted.tif`.
    
    *   **Method:** Select **Moore et al. (1991)**.
    
    *   **LS Factor:** Save output as `ls_factor.tif`.
    
    *   Click **Run**.

2.  **Compile the Final Soil Loss Grid:**
    
    *   Ensure your $R, K, C$, and $P$ factor rasters are loaded (projected to UTM Zone 45N and matching the resolution of the LS factor raster).
    
    *   Open **Raster** > **Raster Calculator...**.
    
    *   Enter the multiplicative RUSLE equation:
        
        `"R_factor@1" * "K_factor@1" * "ls_factor@1" * "C_factor@1" * "P_factor@1"`
    
    *   Save the output as `annual_soil_loss.tif` in your project folder. Click **OK**.

3.  **Classify and Visualize Erosion Severity:**
    
    *   Open the Layer Styling Panel for `annual_soil_loss.tif`.
    
    *   Change Render Type to **Singleband pseudocolor**.
    
    *   Configure an **Equal Interval** classification with 5 classes to categorize soil loss rates:
        *   **Class 1 ($0 \text{ to } 5\text{ t/ha/yr}$):** Low Erosion (Slight).
        *   **Class 2 ($5 \text{ to } 10\text{ t/ha/yr}$):** Moderate.
        *   **Class 3 ($10 \text{ to } 20\text{ t/ha/yr}$):** High.
        *   **Class 4 ($20 \text{ to } 40\text{ t/ha/yr}$):** Very High.
        *   **Class 5 ($> 40\text{ t/ha/yr}$):** Severe (requiring immediate biological terracing or catchment bioengineering).
