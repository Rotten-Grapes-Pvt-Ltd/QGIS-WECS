# Multi-Temporal Workflows

Multi-temporal workflows focus on processing series of satellite images to map environmental dynamics.

---

## 1. River Migration Analysis

* **Goal:** Model how a river channel shifts over time.

* **Workflow:**

  1. Calculate NDWI for Landsat scenes across multiple years.

  2. Reclassify to binary water masks.

  3. Convert rasters to vector centerlines.

  4. Compare centerline offsets to calculate bank migration rates.

---

## 2. Raster Differencing for Sedimentation

* Subtracting bathymetric raster grids of a reservoir across different years to calculate the spatial distribution and volume of accumulated sediment.
