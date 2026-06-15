# Introduction to Time-Series Analysis

Time-series analysis in GIS involves analyzing a sequence of spatial datasets collected over time. This allows us to track environmental change, detect trends, and predict future conditions.

---

## 1. Temporal Datasets

* **Multi-Temporal Imagery:** Satellite data collected at regular intervals over years (e.g., comparing Landsat images from 1990, 2000, 2010, and 2020).

* **Climate Grids:** Daily precipitation or temperature grids (like CHIRPS or ERA5) used to analyze seasonal patterns and climate change trends.

---

## 2. Change Detection Concepts

* **Post-Classification Comparison:** Classifying two images from different dates independently and then overlaying the results to map where land cover has transitioned (e.g., forest converted to agriculture).

* **Raster Differencing:** Subtracting one continuous grid from another (e.g., subtracting a baseline DEM from a post-monsoon DEM to map soil erosion or deposition zones).
