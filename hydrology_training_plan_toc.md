# 5-Day Training Plan — Hydrological Modelling using Geospatial and Remote Sensing Data

# Day 1 — Spatial Thinking, GIS Foundations & Hydrological Context

## Theme
Understanding how spatial data supports hydrological and environmental decision-making.

## Final TOC — Day 1

### 1. Introduction to GIS and Spatial Thinking
- What is GIS?
- Components of GIS
- GIS vs CAD vs MIS vs Excel
- Spatial thinking in hydrology
- Real-world applications in river basin management

### 2. Role of Geospatial Technologies in Water Resource Management
- Watershed planning
- Flood monitoring
- Reservoir management
- Drought assessment
- River morphology studies
- Environmental monitoring

### 3. Spatial Data Models
- Vector data
  - Point
  - Line
  - Polygon
- Raster data
- Continuous vs discrete datasets
- Choosing the right data model

### 4. Understanding Geospatial Datasets
- Administrative boundaries
- Drainage networks
- DEMs
- Satellite imagery
- Land use/land cover
- Rainfall datasets

### 5. Coordinate Reference Systems and Projections
- Geographic CRS
- Projected CRS
- WGS84
- UTM
- EPSG codes
- Reprojection concepts
- Common CRS issues in projects

### 6. Map Scale, Resolution and Accuracy
- Spatial resolution
- Temporal resolution
- Spectral resolution
- Accuracy vs precision
- Scale dependency

### 7. Open Geospatial Ecosystem
- Open-source GIS ecosystem
- OSGeo overview
- Role of:
  - QGIS
  - GDAL
  - PostGIS
  - GeoServer
  - PROJ

### 8. Geospatial Data Formats
- Shapefile limitations
- GeoPackage
- GeoJSON
- GeoTIFF
- CSV with coordinates

### 9. Practical Session
#### Hands-on Exercises
- Loading vector/raster data
- Exploring attributes
- Measuring distances
- CRS transformation
- Basic visualization
- Creating project structure

### 10. Mini Assignment
- Create a watershed overview map using provided datasets

---

# Day 2 — Remote Sensing & Earth Observation for Hydrology

## Theme
Understanding Earth Observation datasets and preparing satellite data for hydrological workflows.

## Final TOC — Day 2

### 1. Introduction to Remote Sensing
- Principles of remote sensing
- Electromagnetic spectrum
- Active vs passive sensors
- Optical vs SAR imagery

### 2. Earth Observation Missions and Sensors
- Sentinel-1
- Sentinel-2
- Landsat
- MODIS
- SRTM
- CHIRPS rainfall

### 3. Understanding Satellite Data Characteristics
- Spatial resolution
- Spectral resolution
- Temporal resolution
- Radiometric resolution

### 4. Satellite Bands and Composites
- RGB composites
- False color composites
- NIR and SWIR concepts
- Hydrology-relevant bands

### 5. Accessing Open Satellite Data
- Copernicus Data Space
- USGS EarthExplorer
- NASA Earthdata
- Open data policies

### 6. Satellite Data Preparation Workflows
- Downloading datasets
- Data organization
- Clipping AOI
- Reprojection
- Resampling
- Raster alignment
- Handling NoData

### 7. DEMs and Terrain Data
- DEM concepts
- Elevation datasets
- Terrain representation
- DEM limitations

### 8. Remote Sensing Applications in Hydrology
- Surface water monitoring
- Flood mapping
- Vegetation monitoring
- Snow cover
- Land degradation

### 9. Practical Session
#### Hands-on Exercises
- Download Sentinel-2 imagery
- Create FCC visualization
- Prepare DEM
- Clip watershed AOI
- Create raster stack

### 10. Mini Assignment
- Prepare multi-layer remote sensing dataset for hydrological analysis

---

# Day 3 — QGIS Workflows & Spatial Data Management

## Theme
Building operational GIS workflows using QGIS and open geospatial tools.

## Final TOC — Day 3

### 1. Introduction to QGIS
- QGIS interface
- Panels and toolbars
- Browser panel
- Processing toolbox
- Project management

### 2. Layer Management and Styling
- Symbology
- Categorized styling
- Graduated styling
- Labels
- Rule-based styling

### 3. Attribute Data and Table Operations
- Attribute tables
- Field calculations
- Data filtering
- Selection tools
- Joins and relations

### 4. Vector Geoprocessing
- Buffer
- Clip
- Dissolve
- Merge
- Union
- Intersect

### 5. Raster Processing
- Raster calculator
- Reclassification
- Mosaic
- Raster clipping
- Terrain visualization

### 6. Spatial Queries and Analysis
- Spatial joins
- Nearest neighbor analysis
- Overlay analysis
- Zonal statistics

### 7. Geospatial Data Management
- Folder structures
- Metadata
- Naming conventions
- Data quality
- Version management

### 8. Web GIS and OGC Services
- WMS
- WMTS
- XYZ tiles
- WFS
- Loading online services in QGIS

### 9. Introduction to Spatial Databases
- Why databases matter
- PostGIS overview
- Files vs databases
- Basic spatial SQL concepts

### 10. QGIS Plugins and Extensions
- QuickMapServices
- Profile Tool
- Semi-Automatic Classification Plugin

### 11. Practical Session
#### Hands-on Exercises
- Watershed analysis workflow
- River buffer analysis
- Settlement proximity analysis
- Raster terrain workflow
- Online basemap integration

### 12. Mini Assignment
- Develop complete GIS project for watershed planning scenario

---

# Day 4 — Terrain Modelling & Hydrological Spatial Analysis

## Theme
Applying GIS and terrain analysis techniques for hydrological modelling and watershed studies.

## Final TOC — Day 4

### 1. Introduction to Terrain Analysis
- Terrain representation
- DEM fundamentals
- Surface modelling concepts

### 2. DEM Processing Workflows
- Sink filling
- DEM conditioning
- Terrain correction
- Raster preprocessing

### 3. Terrain Derivatives
- Slope
- Aspect
- Hillshade
- Curvature
- Elevation profiles

### 4. Hydrological Terrain Analysis
- Flow direction
- Flow accumulation
- Stream extraction
- Watershed delineation
- Drainage network generation

### 5. River Basin and Watershed Analysis
- Basin hierarchy
- Watershed boundaries
- Catchment analysis
- Stream order concepts

### 6. Raster-Based Hydrological Modelling Concepts
- Surface runoff
- Water flow modelling
- Erosion susceptibility
- Terrain influence on hydrology

### 7. Rainfall and Environmental Analysis
- Rainfall rasters
- Zonal statistics
- Raster summarization
- Watershed statistics

### 8. Flood and Water Resource Applications
- Flood-prone area identification
- Reservoir catchment analysis
- Watershed prioritization

### 9. Practical Session
#### Hands-on Exercises
- Generate slope and aspect
- Extract drainage network
- Delineate watershed
- Generate hydrological outputs
- Terrain interpretation

### 10. Mini Assignment
- Perform watershed delineation and terrain analysis for provided basin

---

# Day 5 — Time Series Analysis, Spectral Indices & Decision Support

## Theme
Using temporal satellite analysis and geospatial outputs for environmental monitoring and decision support.

## Final TOC — Day 5

### 1. Introduction to Time-Series Analysis
- Temporal datasets
- Change detection concepts
- Seasonal analysis
- Trend interpretation

### 2. Spectral Indices
- NDVI
- NDWI
- NDBI
- SAVI
- Water and vegetation interpretation

### 3. Environmental Monitoring Applications
- Vegetation health
- Surface water monitoring
- Drought assessment
- Land degradation
- Flood extent mapping

### 4. Multi-Temporal Analysis Workflows
- Comparing satellite dates
- Change detection
- Raster differencing
- Temporal visualization

### 5. Geospatial Decision Support Concepts
- GIS in planning
- Evidence-based decision making
- Spatial indicators
- Environmental reporting

### 6. Cartography and Reporting
- Layout manager
- Map composition
- Legends
- North arrows
- Scale bars
- Exporting maps

### 7. Workflow Automation Concepts
- Batch processing
- Graphical model builder
- Repeatable GIS workflows

### 8. AI-Assisted GIS Workflows
- Using AI for GIS troubleshooting
- Generating expressions
- Documentation support
- Workflow assistance

### 9. Final Integrated Practical
#### Capstone Exercise
Participants perform:
- data preparation
- terrain analysis
- spectral index generation
- temporal interpretation
- final map creation

### 10. Participant Presentation and Discussion
- Group outputs
- Interpretation discussion
- Institutional applicability

### 11. Closing Session
- Key takeaways
- Open-source learning resources
- Further learning roadmap
- Discussion and feedback
