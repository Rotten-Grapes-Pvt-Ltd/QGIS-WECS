# Raster Processing and Map Algebra

Raster processing involves manipulating pixel grids. Because rasters represent continuous surfaces, analysis is executed on a cell-by-cell basis.

---

## 1. The Raster Calculator
The **Raster Calculator** performs math equations on one or more raster grids:

* **Cell-by-Cell Math:**
  $$\text{Output Pixel} = f(\text{Input Pixel}_A, \text{Input Pixel}_B)$$

* **Slope Calculation Example:** Converting elevation data to slope values.

* **Difference Grids:** Subtracting a historic DEM from a modern DEM to estimate reservoir sediment volumes.

---

## 2. Raster Reclassification
Reclassification groups pixel values into thematic categories.

* **Example:**
  ```text
  Pixel values 0 - 10   --> Reclassified to 1 (Low Slope)
  Pixel values 10 - 25  --> Reclassified to 2 (Moderate Slope)
  Pixel values 25 - 90  --> Reclassified to 3 (Steep Slope)
  ```
