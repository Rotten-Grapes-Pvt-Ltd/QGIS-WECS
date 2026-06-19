---
hide:
  - navigation
  - toc
---

<style>
.training-body {
  font-family: 'Outfit', 'Inter', sans-serif;
  color: var(--md-default-fg-color, #1e293b);
}
.training-hero {
  background: linear-gradient(135deg, #0f2027 0%, #1c3643 50%, #203a43 100%);
  color: #ffffff;
  padding: 50px 35px;
  border-radius: 16px;
  margin-bottom: 40px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.12);
  text-align: center;
}
.training-hero h1 {
  font-family: 'Outfit', 'Inter', sans-serif;
  font-size: 2.5rem;
  font-weight: 700;
  margin-top: 0;
  margin-bottom: 12px;
  letter-spacing: -0.5px;
  color: #ffffff !important;
  border-bottom: none !important;
}
.training-hero p {
  font-size: 1.15rem;
  opacity: 0.95;
  max-width: 800px;
  margin: 0 auto;
  line-height: 1.6;
}
.section-title {
  font-family: 'Outfit', 'Inter', sans-serif;
  font-size: 1.7rem;
  font-weight: 600;
  color: var(--md-default-fg-color, #0f172a);
  margin-top: 45px;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  gap: 12px;
  border-bottom: 2px solid var(--md-default-fg-color--lightest, #e2e8f0) !important;
  padding-bottom: 10px;
}
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 24px;
  margin-bottom: 30px;
}
.card {
  background: var(--md-default-bg-color, #ffffff);
  border: 1px solid var(--md-default-fg-color--lightest, rgba(0, 0, 0, 0.08));
  border-radius: 14px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.02);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  flex-direction: column;
}
.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 24px rgba(0,0,0,0.06);
  border-color: var(--md-accent-fg-color, #1565c0);
}
.card-icon {
  font-size: 2.2rem;
  margin-bottom: 15px;
  line-height: 1;
}
.card h3 {
  font-size: 1.25rem;
  font-weight: 650;
  margin-top: 0;
  margin-bottom: 10px;
  color: var(--md-default-fg-color, #0f172a);
  border: none !important;
}
.card p {
  font-size: 0.95rem;
  color: var(--md-default-fg-color--light, #475569);
  line-height: 1.55;
  margin: 0;
}
.split-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
  margin-bottom: 40px;
}
@media (max-width: 768px) {
  .split-container {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}
.highlight-panel {
  border-radius: 16px;
  padding: 25px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.01);
  height: 100%;
  box-sizing: border-box;
}
.panel-blue {
  background: rgba(21, 101, 192, 0.03);
  border: 1px solid rgba(21, 101, 192, 0.12);
}
.panel-green {
  background: rgba(43, 138, 43, 0.03);
  border: 1px solid rgba(43, 138, 43, 0.12);
}
.panel-title {
  font-size: 1.3rem;
  font-weight: 650;
  margin-top: 0;
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.panel-blue .panel-title { color: #1565c0; }
.panel-green .panel-title { color: #2e7d32; }

.timeline {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-top: 25px;
  margin-bottom: 40px;
}
.timeline-item {
  display: flex;
  gap: 24px;
  background: var(--md-default-bg-color, #ffffff);
  border: 1px solid var(--md-default-fg-color--lightest, rgba(0, 0, 0, 0.08));
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.02);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  text-decoration: none !important;
  color: inherit !important;
}
.timeline-item:hover {
  transform: translateX(8px);
  border-color: var(--md-accent-fg-color, #00838f);
  box-shadow: 0 12px 24px rgba(0,0,0,0.06);
  background: rgba(0, 131, 143, 0.01);
}
.day-badge {
  background: linear-gradient(135deg, #1565c0 0%, #00838f 100%);
  color: white;
  padding: 10px 18px;
  border-radius: 10px;
  font-size: 0.9rem;
  font-weight: 700;
  height: fit-content;
  min-width: 80px;
  text-align: center;
  align-self: center;
  box-shadow: 0 4px 8px rgba(21,101,192,0.12);
  letter-spacing: 0.5px;
}
.timeline-content {
  flex: 1;
}
.timeline-content h3 {
  font-size: 1.3rem;
  font-weight: 650;
  margin-top: 0;
  margin-bottom: 8px;
  color: var(--md-default-fg-color, #0f172a);
  border: none !important;
}
.timeline-content p {
  margin: 0;
  color: var(--md-default-fg-color--light, #475569);
  font-size: 0.98rem;
  line-height: 1.55;
}
</style>

<div class="training-body">

<div class="training-hero">
  <h1>Hydrological Modelling Training Plan</h1>
  <p>A 5-Day capacity building technical curriculum compiled and delivered by <strong>Rotten Grapes Pvt. Ltd.</strong> in collaboration with the <strong>Water and Energy Commission Secretariat (WECS)</strong>, Government of Nepal.</p>
</div>

<div class="highlight-panel panel-blue" style="margin-bottom: 40px;">
  <div class="panel-title">Program Mission</div>
  <p style="margin: 0; line-height: 1.65; color: var(--md-default-fg-color, #334155); font-size: 1.05rem;">
    This training program bridges the gap between raw spatial observations and evidence-based water resource management policies. By training hydrologists, drainage engineers, and GIS analysts to utilize modern geospatial pipelines, we enable advanced catchment estimations, soil erosion mapping, and database cataloging for sustainable regional decisions.
  </p>
</div>

<h2 class="section-title">Core Technology Stack</h2>

<div class="grid-container">
  <div class="card">
    <h3>QGIS Desktop</h3>
    <p>The centralized GUI environment for spatial layers, cartography, geoprocessing algorithms, and map layout composition.</p>
  </div>
  <div class="card">
    <h3>PostGIS Database</h3>
    <p>Spatial database client used to construct relational schemas, host vector layers, and run server-side spatial SQL geoprocesses.</p>
  </div>
  <div class="card">
    <h3>Processing Engines</h3>
    <p>SAGA, GRASS, and WhiteboxTools integrated within QGIS to compute terrain derivatives, fill depressions, and extract stream networks.</p>
  </div>
  <div class="card">
    <h3>SWAT+ / QSWAT+</h3>
    <p>Graphical environment for the Soil & Water Assessment Tool (SWAT) to model sediment transport and watershed water balance.</p>
  </div>
</div>

<h2 class="section-title">Hydrological Datasets</h2>

<div class="grid-container">
  <div class="card">
    <h3>Elevation Grids</h3>
    <p>High-resolution Digital Elevation Models (DEMs) including SRTM (30m) and ALOS PALSAR (12.5m) to trace flow paths and slopes.</p>
  </div>
  <div class="card">
    <h3>Vector Layers</h3>
    <p>Catchment boundary shapes, river centerlines, and administrative zones sourced from Natural Earth and national databases.</p>
  </div>
  <div class="card">
    <h3>Satellite Imagery</h3>
    <p>Multispectral bands from Sentinel-2 and Landsat 8/9 to compute indexes (NDWI, NDVI) and perform land cover classifications.</p>
  </div>
  <div class="card">
    <h3>Precipitation Data</h3>
    <p>Point-gauge measurements and gridded CHIRPS datasets to interpolate rain distribution patterns and catchment rainfall volumes.</p>
  </div>
</div>

<div class="split-container">
  <div class="highlight-panel panel-blue">
    <div class="panel-title">Course Pre-requisites</div>
    <p style="margin: 0; color: var(--md-default-fg-color, #334155); font-size: 1rem; line-height: 1.65;">
      Participants should have basic computer operational skills (managing folder hierarchies, relative file pathing, and zip extractions). Elementary knowledge of coordinate refercing systems (CRS) and water basin structures is recommended. QGIS 3.34 LTR must be pre-installed.
    </p>
  </div>
  <div class="highlight-panel panel-green">
    <div class="panel-title">Expected Outcomes</div>
    <p style="margin: 0; color: var(--md-default-fg-color, #334155); font-size: 1rem; line-height: 1.65;">
      Upon completion, attendees will be qualified to model terrain slopes, execute automated stream extractions, calculate zonal stats, compute soil loss indexes, stream remote OGC layers, build local PostGIS tables, and export print-composer maps.
    </p>
  </div>
</div>

<h2 class="section-title">5-Day Curriculum & Modules</h2>

<div class="timeline">
  <a href="day1/index.md" class="timeline-item">
    <div class="day-badge">DAY 1</div>
    <div class="timeline-content">
      <h3>Spatial Thinking & GIS</h3>
      <p>Coordinate reference systems, geographic vs. projected grids, spatial models (vector/raster), scaling, and CRS alignment exercises.</p>
    </div>
  </a>
  <a href="day2/index.md" class="timeline-item">
    <div class="day-badge">DAY 2</div>
    <div class="timeline-content">
      <h3>Remote Sensing & Earth Observation</h3>
      <p>Satellite missions (Sentinel-2, Landsat, SRTM), bands and band combinations, cloud-based data downloads, and terrain derivatives.</p>
    </div>
  </a>
  <a href="day3/index.md" class="timeline-item">
    <div class="day-badge">DAY 3</div>
    <div class="timeline-content">
      <h3>QGIS Workflows & Data Management</h3>
      <p>Workspace setup, symbology rules, table aggregations, vector overlay tools, OGC web streaming, and local PostGIS server deployments.</p>
    </div>
  </a>
  <a href="day4/index.md" class="timeline-item">
    <div class="day-badge">DAY 4</div>
    <div class="timeline-content">
      <h3>Terrain & Hydrological Analysis</h3>
      <p>DEM filling, flow accumulation, channel network extractions, basin delineation, and RUSLE soil erosion vulnerability modeling.</p>
    </div>
  </a>
  <a href="day5/index.md" class="timeline-item">
    <div class="day-badge">DAY 5</div>
    <div class="timeline-content">
      <h3>Time Series & Decision Support</h3>
      <p>Multi-temporal NDWI tracking, water extent change mapping, print Composer layouts, automation builders, and final project submissions.</p>
    </div>
  </a>
</div>

</div>
