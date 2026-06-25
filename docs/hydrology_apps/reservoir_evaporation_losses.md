# Reservoir Evaporation Losses

Reservoir evaporation is a major component of a basin's water budget, representing a direct volumetric loss of stored surface water. Quantifying evaporation losses relative to direct precipitation inputs is critical for reservoir management, water availability forecasting, and calculating net storage changes.

> [!TIP]
> **Data Sources & Acquisition:**
> To calculate monthly reservoir evaporation budgets, obtain the following free global datasets:
> 
> *   **CHIRPS Daily Precipitation CSV:** Acquire daily precipitation time series in CSV format from the [Climate Engine Portal](https://app.climateengine.org/climateEngine). In the Climate Engine interface, select the CHIRPS dataset, choose "Precipitation" as the variable, draw a polygon over your target reservoir basin, select the daily temporal resolution, and export the time series as a CSV.
> 
> *   **MODIS MOD16A2GF Monthly Evapotranspiration:** Download monthly actual evapotranspiration (ET) grids (500m resolution) from the [NASA AppEEARS Portal](https://appeears.earthdatacloud.nasa.gov/). Start a new point or area extraction request, search for the `MOD16A2GF.061` (MODIS/Terra Net Evapotranspiration Gap-Filled Monthly) product, select your reservoir's coordinates and desired date range, and download the output dataset.

---

## 1. Core Objectives

*   **Process** daily satellite precipitation records into monthly equivalents using Python.

*   **Integrate** daily/monthly rainfall and monthly evapotranspiration datasets into a unified Excel database.

*   **Delineate** the reservoir's water surface area polygon at a specific elevation contour (114 meters) using digital elevation model (DEM) data in QGIS.

*   **Convert** rainfall and evapotranspiration depths ($mm$) to volumetric Million Cubic Meters ($MCM$) to audit the net reservoir water budget.

---

## 2. Key GIS and Tabular Inputs

*   **Daily Precipitation Time Series:** CHIRPS daily CSV file (`chirps_daily.csv`).

*   **Monthly Evapotranspiration Data:** MODIS MOD16A2GF monthly raster grids or extracted coordinates table (`modis_monthly_et.csv` or raster).

*   **Digital Elevation Model (DEM):** Projected topographic DEM of the reservoir basin area (`reservoir_dem_utm.tif`).

---

## 3. Step-by-Step Reservoir Evaporation Losses Workflow

Calculating reservoir water loss requires tabular preprocessing, spatial contour-based boundary delineation, and volumetric conversion calculations:

```text
       [ CHIRPS Daily Precipitation CSV ]          [ MODIS Monthly ET Product ]
                      │                                         │
                      ▼ (Python Pandas Script)                  │
       [ Monthly Aggregated Precipitation ]                     │
                      │                                         │
                      └─────────────────┬───────────────────────┘
                                        ▼ (Python Pandas Merge)
                         [ Excel Database: P vs. ET ]
                                        │
                                        ▼ (SAGA Contour & Delineation @ 114m)
                       [ Reservoir Surface Area (m²) ]
                                        │
                                        ▼ (Volumetric MCM Conversions)
                       [ Volumetric Comparison Matrix ]
```

### Step 1: Aggregate Daily Precipitation to Monthly (Python)

To compare daily precipitation with monthly MODIS evapotranspiration, we must aggregate the daily CHIRPS data. We will write a Python script using the `pandas` library to calculate monthly totals and averages:

```python
import pandas as pd

def process_chirps_data(csv_path, output_path):
    # 1. Load the CHIRPS daily CSV
    df = pd.read_csv(csv_path)
    
    # 2. Convert date column to datetime
    # Update 'Date' to match the column name in your Climate Engine CSV (e.g. 'date' or 'time')
    df['Date'] = pd.to_datetime(df['Date'])
    
    # 3. Extract Year and Month for grouping
    df['Year'] = df['Date'].dt.year
    df['Month'] = df['Date'].dt.month
    
    # 4. Group data to calculate monthly sum (total depth) and daily average
    # Update 'precipitation' to match your CSV's rainfall column name (e.g. 'precip', 'value', or 'P')
    monthly_data = df.groupby(['Year', 'Month']).agg(
        Precip_Total_mm=('precipitation', 'sum'),
        Precip_Daily_Avg_mm=('precipitation', 'mean')
    ).reset_index()
    
    # 5. Create a clean date representation for the monthly record
    monthly_data['Date'] = pd.to_datetime(monthly_data[['Year', 'Month']].assign(Day=1))
    
    # 6. Reorder and export to CSV
    monthly_output = monthly_data[['Date', 'Year', 'Month', 'Precip_Total_mm', 'Precip_Daily_Avg_mm']]
    monthly_output.to_csv(output_path, index=False)
    print(f"Successfully processed daily records to: {output_path}")

# Run the function
process_chirps_data('chirps_daily.csv', 'chirps_monthly_aggregated.csv')
```

---

### Step 2: Create a Unified Comparison Excel Sheet (Python)

To perform direct comparison calculations, merge the monthly CHIRPS precipitation and MODIS ET datasets using a unified date field:

```python
import pandas as pd

def merge_p_and_et(chirps_monthly_path, modis_monthly_path, excel_output_path):
    # Load processed monthly CHIRPS precipitation
    df_p = pd.read_csv(chirps_monthly_path)
    df_p['Date'] = pd.to_datetime(df_p['Date'])
    
    # Load monthly MODIS ET dataset
    df_et = pd.read_csv(modis_monthly_path)
    df_et['Date'] = pd.to_datetime(df_et['Date'])
    
    # Merge datasets on the Date column
    # Assumes 'et_mm' represents the monthly ET depth in the MODIS CSV
    df_merged = pd.merge(df_p, df_et[['Date', 'et_mm']], on='Date', how='inner')
    
    # Rename columns for clarity
    df_merged.rename(columns={'Precip_Total_mm': 'Rainfall_mm', 'et_mm': 'Evaporation_mm'}, inplace=True)
    
    # Write to Excel
    df_merged.to_excel(excel_output_path, sheet_name='P_vs_ET_Comparison', index=False)
    print(f"Excel database created successfully: {excel_output_path}")

# Run the merge
merge_p_and_et('chirps_monthly_aggregated.csv', 'modis_monthly_et.csv', 'reservoir_water_budget.xlsx')
```

---

### Step 3: Delineate Reservoir Surface Area at 114m Elevation Contour

To estimate volumetric water gains and losses, we must extract the reservoir's surface area ($A$) corresponding to a specific water level height. In this tutorial, the maximum operating pool level contour is defined at **114 meters**.

*   **What We Are Doing:** Extracting the 114m topographic contour line from the DEM and converting it into a closed polygon to represent the reservoir water surface area.

*   **Why This Step is Needed:** Evaporation and precipitation volumes are functions of surface area. Finding the area corresponding to the 114m contour represents the active reservoir pool boundary at that target elevation.

*   *Input:* Projected topographic DEM (`reservoir_dem_utm.tif`).

*   *Output:* Closed reservoir vector polygon (`reservoir_114m_boundary.gpkg`) and area attribute.

*   *How to Extract Contours in QGIS:*
    
    *   **Tool Path:** Go to main menu **Raster** > **Extraction** > **Contour...**.
    
    *   **Input Layer:** Select `reservoir_dem_utm.tif`.
    
    *   **Interval between contour lines:** Set to `1.0` meter (or set to `0.0` if using a single contour tool).
    
    *   **Result:** Save as `basin_contours.gpkg`.
    
    *   *Alternative SAGA Contour Tool:* Open the **Processing Toolbox** and search for **SAGA** > **Vector <-> Grid** > **Contour Lines from Grid**. Set the input grid, configure a 1-meter interval, and run.

*   *Extract the 114m Boundary Polygon:*
    
    *   Open the attribute table of `basin_contours.gpkg`.
    
    *   Filter/select the contour line where the elevation field value equals `114`. Export the selected line as `contour_114m.gpkg`.
    
    *   Convert this open/closed contour line into a polygon. Go to main menu **Vector** > **Geometry Tools** > **Lines to Polygons...**. Select `contour_114m.gpkg` as input and save the result as `reservoir_114m_boundary.gpkg`.
    
    *   Open the attribute table of `reservoir_114m_boundary.gpkg`, open **Field Calculator**, create a new decimal field named `Area_m2`, and calculate the geometry area using the expression:
        `$area`
        *(Record this surface area value; for this example, let's assume the calculated area is **8,500,000 m²**).*

---

### Step 4: Volumetric Conversion to Million Cubic Meters (MCM)

To audit the net volume of water entering the reservoir via precipitation and leaving via evaporation, convert depth values ($mm$) to Million Cubic Meters ($MCM$).

#### The Conversion Formula:
$$\text{Volume } (m^3) = \text{Area } (m^2) \times \frac{\text{Depth } (mm)}{1,000}$$

$$\text{Volume } (MCM) = \frac{\text{Volume } (m^3)}{1,000,000} = \text{Area } (m^2) \times \text{Depth } (mm) \times 10^{-9}$$

#### Step-by-Step Computational Example:
Assuming a delineated reservoir surface area of **$8,500,000\text{ m}^2$** (at the 114m pool contour):

1.  **Volumetric Rainfall (MCM):**
    If the monthly CHIRPS precipitation depth is **$120\text{ mm}$**:
    $$\text{Rainfall Volume (MCM)} = 8,500,000 \times 120 \times 10^{-9} = 1.02\text{ MCM}$$

2.  **Volumetric Evaporation Loss (MCM):**
    If the monthly MODIS ET depth is **$145\text{ mm}$**:
    $$\text{Evaporation Volume (MCM)} = 8,500,000 \times 145 \times 10^{-9} = 1.2325\text{ MCM}$$

3.  **Net Reservoir Water Balance (MCM):**
    $$\text{Net Balance (MCM)} = \text{Rainfall Volume} - \text{Evaporation Volume}$$
    $$\text{Net Balance (MCM)} = 1.02 - 1.2325 = -0.2125\text{ MCM}$$
    *(A negative net balance indicates the reservoir lost $0.2125\text{ MCM}$ of water to the atmosphere, requiring catchment streamflow runoff inputs to maintain storage levels).*

---

## 4. Multi-Temporal Evaporation Budget Comparison Table

Compile a monthly water balance sheet in your Excel database to analyze the seasonal trends of direct rainfall gain versus evaporation loss:

| Date | Reservoir Area ($m^2$) | Rainfall ($mm$) | Evaporation ($mm$) | Vol. Rainfall ($MCM$) | Vol. Evaporation ($MCM$) | Net Atmospheric Budget ($MCM$) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Jan 2026** | 8,500,000 | 15.0 | 95.0 | 0.128 | 0.808 | **-0.680 (Net Loss)** |
| **Feb 2026** | 8,500,000 | 25.0 | 110.0 | 0.213 | 0.935 | **-0.722 (Net Loss)** |
| **Mar 2026** | 8,500,000 | 45.0 | 125.0 | 0.383 | 1.063 | **-0.680 (Net Loss)** |
| **Apr 2026** | 8,500,000 | 110.0 | 130.0 | 0.935 | 1.105 | **-0.170 (Net Loss)** |
| **May 2026** | 8,500,000 | 250.0 | 120.0 | 2.125 | 1.020 | **+1.105 (Net Gain)** |
| **Jun 2026** | 8,500,000 | 320.0 | 90.0 | 2.720 | 0.765 | **+1.955 (Net Gain)** |

---

## 5. Hydrological and Engineering Significance

*   **Dry Season Storage Management:** Identifies months where high evaporative demands exhaust reservoir storage, allowing operators to plan releases for domestic supply and irrigation before severe deficits manifest.

*   **Water Productivity Analysis:** Enables engineers to contrast irrigation consumption benefits against open-water evaporative losses, optimizing cropping patterns and water release scheduling.

*   **Climate Change Vulnerability:** Analyzes how rising ambient temperatures and enhanced ET affect the long-term yield capacity and safe operational storage of reservoirs.
