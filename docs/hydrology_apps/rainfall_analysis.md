# Rainfall Spatial Analysis

Rainfall is the primary input driving watershed hydrological cycles. Because precipitation varies across space and time, spatial analysis in GIS is required to convert point observations (rain gauges) or coarse satellite grids into localized precipitation volumes over a catchment.

> [!TIP]
> **Data Sources & Acquisition:**
> For hydrological analysis without in-situ weather station records, utilize these free global gridded precipitation repositories:
> 
> *   **CHIRPS Precipitation (Daily/Monthly, 0.05° resolution):** Specifically developed for agricultural drought and watershed monitoring. Download raw GeoTIFF grids directly from the [UCSB Climate Hazards Center CHIRPS Directory](https://www.chc.ucsb.edu/data/chirps) or access it dynamically through [Climate Engine](https://climateengine.com/).
> 
> *   **GPM IMERG (Half-hourly/Daily, 0.1° resolution):** High-precision satellite precipitation estimates from NASA. Download in NetCDF or HDF5 format from [NASA Earthdata Search](https://search.earthdata.nasa.gov/) by filtering for "GPM IMERG".
> 
> *   **ERA5-Land Reanalysis (Hourly/Monthly, 9km resolution):** Long-term historical weather reanalysis grids back to 1950. Download from the [Copernicus Climate Data Store (CDS)](https://cds.climate.copernicus.eu/).
> 
> *   **Global Historical Weather Stations (GHCN):** Ground rain gauge observations. Download historical station summaries for free from the [NOAA National Centers for Environmental Information (NCEI)](https://www.ncei.noaa.gov/products/land-based-station/global-historical-climatology-network).

---

## 1. Core Objectives

*   **Plot** rain gauge station locations from coordinates and attribute data.

*   **Interpolate** discrete point rain gauge measurements to create a continuous rainfall surface.

*   **Delineate** gauge influence zones to allocate representative weights to stations.

*   **Extract** catchment-wide Mean Areal Precipitation (MAP) values from grids.

---

## 2. Key GIS Inputs

*   **Precipitation Tables:** Spreadsheet/CSV containing station names, coordinates (Lat/Lon), and rainfall measurements (mm).

*   **Catchment Boundary:** Watershed polygon layer (`watershed.gpkg`).

*   **Gridded Precipitation Products:** (Optional alternative) Coarse satellite rasters (CHIRPS, GPM IMERG).

---

## 3. Step-by-Step Rainfall Analysis Workflow

Catchment-wide rainfall assessment requires converting tabular point gauge records into spatial zones and grids using sequential geoprocessing steps:

![flow_chart](images/rainfall_analysis/flow_chart.png)

1.  **Import and Plot Rain Gauge Station Data (CSV to Points):**
    
    *   **What We Are Doing:** Importing a text file (CSV/Excel) containing rain gauge coordinates and rainfall values, and converting it into a GIS vector point layer.
    
    *   **Why This Step is Needed:** Ground measurements are recorded in tabular spreadsheets. Before QGIS can perform spatial interpolations, these coordinates must be plotted on a map as vector geometry points.
    
    *   *Input:* CSV table with station metadata and rainfall records (e.g., `gauge_data.csv`).
    
    *   *Output:* Vector point layer (e.g., `gauges_wgs84.gpkg`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Go to the main QGIS menu and select **Layer** > **Add Layer** > **Add Delimited Text Layer...**.
        
        *   **File Name:** Select your CSV file (`gauge_data.csv`).
        
        *   **File Format:** Check **CSV (comma separated values)**.
        
        *   **Geometry Definition:** Check **Point coordinates**. Set **X field** to the longitude column (e.g., `Longitude`) and **Y field** to the latitude column (e.g., `Latitude`).
        
        *   **Geometry CRS:** Select `EPSG:4326` (WGS 84).
        
        *   **Output:** Click Add, then right-click the loaded layer > **Export** > **Save Features As...** to save it as `gauges_wgs84.gpkg`.

2.  **Reproject Points to Metric CRS:**
    
    *   **What We Are Doing:** Transforming the coordinate reference system (CRS) of the rain gauges from degrees (geographic) to meters (projected).
    
    *   **Why This Step is Needed:** Distance-decay interpolation formulas (like IDW) calculate weights using horizontal distances. If the layer is in geographic degrees, QGIS will calculate distance in degrees, resulting in distorted interpolations and circular "bull's-eyes". Reprojecting to a metric system (e.g., UTM) ensures that distances are calculated accurately in meters.
    
    *   *Input:* Geographic point layer (`gauges_wgs84.gpkg`) from **Step 1**.
    
    *   *Output:* Projected metric point layer (`gauges_utm.gpkg`).
    
    *   *How to Fill the Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **Vector General** > **Reproject Layer**.
        
        *   **Input Layer:** Select `gauges_wgs84.gpkg`.
        
        *   **Target CRS:** Select your local UTM zone (e.g., `EPSG:32645` for UTM Zone 45N).
        
        *   **Reprojected:** Save the output as `gauges_utm.gpkg`.

3.  **Interpolate Rainfall Surface (SAGA Inverse Distance Weighted or Kriging):**
    
    *   **What We Are Doing:** Generating a continuous raster grid of rainfall values from the discrete gauges using spatial interpolation.
    
    *   **Why This Step is Needed:** Rain gauges only measure rainfall at specific points. We need to estimate rainfall values in the unsampled areas between the stations to represent the spatial distribution of rainfall across the entire catchment.
    
    *   *Input:* Projected point layer (`gauges_utm.gpkg`) from **Step 2**.
    
    *   *Output:* Continuous interpolated rainfall raster (`rainfall_surface.tif`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Interpolation** > **Inverse Distance Weighted**.
        
        *   **Points:** Select `gauges_utm.gpkg`.
        
        *   **Attribute:** Select the field representing precipitation depth (e.g., `Rain_mm`).
        
        *   **Target Grid System:** Select `[0] user defined`.
        
        *   **Cellsize:** Enter a cell size in meters (e.g., `100` for a 100-meter grid resolution).
        
        *   **Grid:** Click and save the output grid as `rainfall_surface.tif`.
        
        *   *Alternative Ordinary Kriging Path:* If geostatistical autocorrelation modeling is preferred, navigate to **SAGA** > **Grid - Interpolation** > **Ordinary Kriging**. Fill in points, attribute, and target cellsize as above.

4.  **Delineate Rain Gauge Influence Zones (SAGA Thiessen Polygons):**
    
    *   **What We Are Doing:** Delineating boundaries around each rain gauge to partition the catchment into polygons of influence (Voronoi cells).
    
    *   **Why This Step is Needed:** The Thiessen Polygon method is the traditional hydrological standard for computing areal precipitation. Every location inside a Thiessen polygon is closer to that polygon's rain gauge than to any other, meaning the gauge's value is assumed to represent that entire polygon zone.
    
    *   *Input:* Projected point layer (`gauges_utm.gpkg`) from **Step 2**.
    
    *   *Output:* Thiessen polygon vector layer (`thiessen_zones.gpkg`).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Vector - Point Tools** > **Thiessen Polygons**.
        
        *   **Points:** Select `gauges_utm.gpkg`.
        
        *   **Frame:** Select **buffer** or select the bounding box of your study area.
        
        *   **Polygons:** Click and save as `thiessen_zones.gpkg`.
        
        *   *Post-Processing:* Use **Vector** > **Geoprocessing Tools** > **Clip** to crop `thiessen_zones.gpkg` with your watershed boundary.

5.  **Extract Mean Areal Precipitation (SAGA Raster Statistics for Polygons):**
    
    *   **What We Are Doing:** Calculating the spatial average precipitation (Mean Areal Precipitation or MAP) over the catchment boundary from the interpolated raster surface or a satellite product.
    
    *   **Why This Step is Needed:** Hydrological models require a single representative precipitation input (in mm) for the catchment to compute runoff. Running zonal statistics extracts this spatial average by summing the cell values within the catchment boundary and dividing by the cell count.
    
    *   *Input:* Watershed polygon boundary and the interpolated raster (`rainfall_surface.tif` from **Step 3**) or a satellite grid (like CHIRPS).
    
    *   *Output:* Attribute table populated with mean rainfall depth (mm) and total volume (m³).
    
    *   *How to Fill the SAGA Form in QGIS:*
        
        *   **Tool Path:** Open the **Processing Toolbox** and navigate to **SAGA** > **Grid - Tools** > **Raster Statistics for Polygons**.
        
        *   **Grid:** Select `rainfall_surface.tif` (or your reprojected CHIRPS/GPM raster).
        
        *   **Polygons:** Select your catchment boundary layer.
        
        *   **Method:** Check the **mean** and **sum** statistic boxes.
        
        *   **Result:** Save the output layer as `catchment_precipitation.gpkg`. The mean rainfall value will be appended directly to the polygon's attribute table.

---

## 4. Analytical Methods Comparison

Hydrologists choose different methods depending on rain gauge density and topographic complexity:

| Method | Best Suited For | Advantages | Limitations |
| :--- | :--- | :--- | :--- |
| **Thiessen Polygons** | Flat terrain, sparse gauge networks | Simple to calculate; assigns objective weights based on geometry | Abrupt transitions at boundaries; ignores elevation effects |
| **Inverse Distance Weighting (IDW)** | Flat to moderate terrain, dense networks | Smooth transitions; easy to compute grid surfaces | Prone to "bull's eye" artifacts; does not account for topography |
| **Kriging (Geostatistical)** | Dense gauge networks, complex terrains | Provides statistical error maps; models spatial trends | Computationally intensive; requires statistical validation |
| **Orographic/Co-Kriging** | Mountainous zones with high relief | Integrates elevation (DEM) to estimate rain increases on slopes | Requires dense historical data to build accurate correlations |

---

## 5. Hydrological Significance

*   **Water Balance Modeling:** Mean Areal Precipitation is the foundational input for water balance equations ($P - ET - Q = \Delta S$, where $P$ is precipitation, $ET$ is evapotranspiration, $Q$ is runoff, and $\Delta S$ is storage change).

*   **Design Storm Isohyetal Maps:** Isohyetal contours generated from interpolated rainfall surfaces outline zones of maximum flood vulnerability, which are critical for sizing spillways, drainage canals, and bridges.

*   **Runoff Hydrographs:** Semi-distributed models (like SWAT) run calculations for each sub-basin using the zonal average rainfall extracted via GIS.
