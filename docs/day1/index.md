# Spatial Thinking, GIS Foundations & Hydrological Context

Welcome to the first module of the **Hydrological Modelling using Geospatial and Remote Sensing Data** training program. This module lays the essential groundwork for all spatial analysis, mapping, and watershed modeling workflows that you will build throughout the week.

We will transition from understanding how spatial data represents the physical environment to working hands-on inside a Geographic Information System (GIS) to process layers and compile professional map layouts.

---

## Learning Objectives
By the end of today's sessions, you will be able to:

* **Distinguish** spatial datasets from traditional relational databases and spreadsheets.

* **Explain** how physical terrain features (elevation, slope, aspect) dictate flow direction and accumulation in river basins.

* **Identify** the differences between Geographic and Projected Coordinate Reference Systems, select the correct UTM zones for Nepal (44N/45N), and avoid projection errors.

* **Select** appropriate spatial data models (Vector vs. Raster) and storage formats (Shapefiles vs. GeoPackages vs. Cloud-Optimized GeoTIFFs) for diverse hydrological tasks.

* **Navigate** the open-source geospatial ecosystem (QGIS, GDAL, PostGIS, GeoServer).

* **Construct** a standardized GIS project directory, import data, reproject layers, style symbology, and generate a print-ready map PDF.

---

## Learning Roadmap
Below is the progression of topics for today, moving from core theoretical concepts to practical, independent map compilation:

```mermaid
flowchart TD
    subgraph Row1 [" "]
        direction LR
        A["1. Spatial Thinking &<br/>GIS Foundations<br/>(McHarg Overlays)"] --> B["2. Data Models &<br/>Coordinate Systems<br/>(Vector/Raster/UTM)"] --> C["3. Open Ecosystem &<br/>File Formats<br/>(QGIS & GPKG/GeoTIFF)"]
    end
    
    style A fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style B fill:#fff3e0,stroke:#d84315,stroke-width:2px
    style C fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style Row1 fill:none,stroke:none,stroke-width:0px
```

---

## Topics and Schedule

* **[Topic 1: Introduction to GIS and Spatial Thinking](01_intro_gis.md)**
  An introduction to the five components of GIS, map overlay concepts, and how spatial adjacency and Tobler's First Law govern water movement.

* **[Topic 2: Role of Geospatial Technologies in Water Resource Management](02_role_geospatial.md)**
  Explores real-world use cases, including watershed planning, flood inundation mapping (SAR), reservoir sedimentation, and drought indexes compliance.

* **[Topic 3: Spatial Data Models and Geospatial Datasets](03_spatial_data_models.md)**
  A deep-dive into vector primitives (points, lines, polygons) versus continuous raster grids, and an overview of key datasets (DEMs, LULC, rainfall, boundaries) and their significance in hydrology.

* **[Topic 4: Coordinate Reference Systems and Projections](04_crs_projections.md)**
  An essential guide to geoids, datums (WGS 84), projections, the UTM coordinate grid, EPSG codes, and how to troubleshoot displaced layers.

* **[Topic 5: Map Scale, Resolution and Accuracy](05_map_scale.md)**
  Details map scales, spatial/temporal/spectral resolutions, the accuracy-precision matrix, and scale-dependent vector generalizations.

* **[Topic 6: Open Geospatial Ecosystem](06_open_geospatial.md)**
  Overview of the OSGeo software stack: QGIS Desktop, GDAL/OGR translators, PostGIS spatial databases, GeoServer, and the PROJ transformation engine.

* **[Topic 7: Geospatial Data Formats](07_geospatial_formats.md)**
  Details legacy Shapefile limitations, OGC GeoPackage advantages, web-native GeoJSON, and Cloud-Optimized GeoTIFF (COG) internal tile mechanisms.

## Existing Resources

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?si=B_QsD40XYlQjaaaY&amp;list=PL3MO67NH2XxLAFn3jc7gOhXLD9YFx-oew" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


<iframe width="560" height="315" src="https://www.youtube.com/embed/id04Bpuk4Fs?si=cOfG_taqgXfoYAvS" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

- <a href="https://ocw.mit.edu/courses/res-str-001-geographic-information-system-gis-tutorial-january-iap-2022/pages/gis-level-1/" target="_blank">MIT OpenCourseware GIS</a>

- <a href="https://www.coursera.org/specializations/gis" target="_blank">Coursera GIS</a>

