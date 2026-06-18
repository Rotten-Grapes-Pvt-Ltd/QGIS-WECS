# Integrated Capstone Practical: River Basin Delineation and Analysis

This capstone laboratory integrates the spatial workflows, geoprocessing techniques, and remote sensing calculations developed throughout the 5-day curriculum.

---

## 1. Project Scenario and Objectives

The Water and Energy Commission Secretariat (WECS) requires an integrated environmental study of the **West Rapti River Basin** in Nepal. Your team is tasked with delineating the catchment area, calculating morphometric parameters, classifying changes in surface water extent, and generating a professional map booklet for policy submission.

---

## 2. Target Deliverables

You must submit a single compressed ZIP file named `WECS_Capstone_Group_[Number].zip` containing:

1.  **Catchment Database (`rapit_basin.gpkg`):**
    
    A GeoPackage containing your delineated watershed boundary, stream network line vectors categorized by Strahler order, and water extent polygons.

2.  **Water Index Calculations:**
    
    A GeoTIFF raster representing NDWI and dry-season surface water.

3.  **PDF Layout Document (`rapti_basin_report.pdf`):**
    
    A professional, publication-ready $A3$ map layout showing the basin elevation, drainage lines, and surface water boundaries with appropriate grids, legend, and inset index map.

---

## 3. Step-by-Step Task Checklist

*   **Task 1: Project Workspace and CRS Setup**
    
    *   Create a directory structure: `/project/data/`, `/project/models/`, `/project/outputs/`.
    
    *   Initialize a new QGIS project. Set the Project Coordinate Reference System (CRS) to **WGS 84 / UTM Zone 44N** (EPSG: `32644`).

*   **Task 2: Catchment Delineation from DEM**
    
    *   Load the raw DEM `dem_raw_rapti.tif`.
    
    *   Run SAGA **Fill Sinks (Wang & Liu)** to remove internal depression artifacts.
    
    *   Compute Flow Direction and Flow Accumulation rasters.
    
    *   Run **Channel Network** and extract a stream network vector layer using a flow threshold of $> 5,000$ pixels.
    
    *   Generate the catchment basin boundary polygon using your coordinate points for the downstream outlet location.

*   **Task 3: Remote Sensing Water Index Analysis**
    
    *   Load pre-processed Sentinel-2 Band 3 (Green) and Band 8 (NIR) rasters.
    
    *   Use the **Raster Calculator** to compute NDWI.
    
    *   Reclassify the NDWI raster to isolate surface water bodies (where NDWI $> 0.0$).
    
    *   Convert the water raster to a vector polygon layer and clip the output to your delineated catchment boundary.

*   **Task 4: Symbology and Spatial Database Compilation**
    
    *   Save all vector outputs (watershed boundary, streams, water polygons) into a single database container `rapti_basin.gpkg`.
    
    *   Apply a graduated line width to the stream vector based on the Strahler stream order attribute.
    
    *   Style the DEM with a singleband pseudocolor ramp using elevation colors.

*   **Task 5: Print Layout Design**
    
    *   Create a new print layout. Set page size to $A3$ Landscape.
    
    *   Add the main map canvas, a legend with edited descriptive labels, a dynamic scale bar, and a north arrow.
    
    *   Add a coordinate grid overlay (UTM grid ticks labeled with unit meters on all borders).
    
    *   Add an inset index map showing the location of the West Rapti Basin inside Nepal.
    
    *   Export the finalized layout as a PDF file.
