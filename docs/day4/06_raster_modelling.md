# Raster-Based Hydrological and Soil Erosion Modelling

Raster grids allow us to model complex spatial environmental processes (such as surface runoff generation and soil erosion rates) by combining multiple spatial datasets. By performing cell-by-cell calculations, the raster layout models how physical variables interact across the entire catchment basin. This section covers peak discharge modeling (the Rational Method), runoff depth modeling (the SCS Curve Number method), and soil erosion susceptibility mapping using the Revised Universal Soil Loss Equation (RUSLE).

---

## 1. Surface Runoff Modeling

Estimating surface runoff volume and peak discharge is critical for designing storm drains, calculating culvert capacity, and modeling floodplain hazard zones.

### The Rational Method (Peak Discharge)

The Rational Method is a simple empirical model used to calculate the peak surface runoff rate ($Q$) at a catchment outlet:

$$Q = C \times I \times A$$

To calculate peak discharge in metric units ($Q$ in $m^3/\text{sec}$), we apply a conversion factor based on the units of input parameters:

$$Q = \frac{C \times I \times A}{3.6 \times 10^6}$$

Where:

*   $Q$ is the peak runoff rate (in cubic meters per second, $m^3/\text{sec}$).

*   $C$ is the dimensionless runoff coefficient, representing the fraction of rainfall that becomes surface runoff.
    
    *   **Paved / Asphalt Areas:** $C = 0.90$ (high runoff)
    
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

### The SCS Curve Number (CN) Method (Runoff Volume)

Developed by the USDA Soil Conservation Service, this model calculates cumulative runoff depth based on land use, soil hydrologic properties, and antecedent soil moisture.

*   **The Runoff Equation:**
    
    $$Q = \frac{(P - I_a)^2}{P - I_a + S} \quad \text{for } P > I_a$$
    
    Where:
    
    *   $Q$ is the cumulative runoff depth (in millimeters, $mm$).
    
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

---

## 2. Soil Erosion Susceptibility (The RUSLE Model)

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
    
    Where:
    
    *   $\alpha$ is the Specific Catchment Area (SCA) from flow accumulation.
    
    *   $\theta$ is the local slope angle in degrees.
    
    *   $m$ and $n$ are empirical exponents ($m \approx 0.4 \text{ to } 0.6$, $n \approx 1.2 \text{ to } 1.3$).

*   **Cover Management Factor ($C$):**
    
    Reflects the vegetative canopy's ability to shield soil from raindrop impact.
    
    Ranges from $0.001$ (dense forest) to $1.0$ (barren soil). Often mapped using satellite-derived NDVI grids:
    
    $$C = \exp \left( -a \times \frac{\text{NDVI}}{\beta - \text{NDVI}} \right)$$

*   **Conservation Support Practice Factor ($P$):**
    
    Reflects the impact of soil conservation practices, such as terracing or contour farming.
    
    In steep Himalayan catchments, active terracing drops the $P$ value from $1.0$ (no support) to $0.1$, significantly reducing modeled soil loss.

---

## 3. Step-by-Step Exercise: Delineating Soil Erosion in QGIS

We will generate an LS factor raster and compile the final soil erosion map.

1.  **Verify Input Rasters:**
    
    Ensure all input factor rasters ($R, K, C, P$) are loaded in QGIS, projected to the same metric coordinate system (e.g., **UTM Zone 44N**), and have matching pixel resolutions.

2.  **Calculate the LS Factor in SAGA:**
    
    Go to **Processing Toolbox** > **SAGA** > **Terrain Analysis - Hydrology** > **LS Factor**.
    
    *   **Elevation:** Select your conditioned, filled DEM `filled_dem.tif`.
    
    *   **Slope:** Select your calculated slope raster `slope_degrees.tif` (or let SAGA calculate it).
    
    *   **Area:** Select your flow accumulation raster `flow_accumulation.tif`.
    
    *   **Method:** Select `Moore et al. (1991)`.
    
    *   **LS Factor:** Save output as `ls_factor.tif`.
    
    Click **Run**.

3.  **Compile Final Soil Erosion Grid:**
    
    Open **Raster** > **Raster Calculator...**.
    
    Enter the multiplicative RUSLE equation:
    
    `"R_factor@1" * "K_factor@1" * "ls_factor@1" * "C_factor@1" * "P_factor@1"`
    
    Save the output as `annual_soil_loss.tif` in your project folder. Click **OK**.

4.  **Categorize Erosion Severity:**
    
    Double-click `annual_soil_loss.tif` and open **Symbology**.
    
    Change Render Type to **Singleband pseudocolor**.
    
    Configure an **Equal Interval** classification with 5 classes to categorize soil loss rates:
    
    *   **Class 1 (0 to 5 t/ha/yr):** Low Erosion (Slight).
    
    *   **Class 2 (5 to 10 t/ha/yr):** Moderate.
    
    *   **Class 3 (10 to 20 t/ha/yr):** High.
    
    *   **Class 4 (20 to 40 t/ha/yr):** Very High.
    
    *   **Class 5 (> 40 t/ha/yr):** Severe (requiring immediate terracing or restoration).
