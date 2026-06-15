# Practical Session: Geoprocessing and Spatial Queries in QGIS

This practical session walks you through applying vector geoprocessing tools (buffering, clipping) and calculating basin statistics using zonal statistics.

---

## 1. Buffer and Clip Operations

1. Load `major_rivers.gpkg` and `nepal_districts.gpkg` into QGIS.

2. Create a $500	ext{ m}$ buffer zone around all major rivers:

   * Go to **Vector** > **Geoprocessing Tools** > **Buffer**.

   * **Input Layer:** `major_rivers.gpkg`.

   * **Distance:** `500` (Make sure your layer is projected in meters).

   * Check **Dissolve result** to merge overlapping buffers.

   * Save the output as `river_buffers_500m.gpkg`.

3. Clip the river buffer zone to a specific district boundary:

   * Go to **Vector** > **Geoprocessing Tools** > **Clip**.

   * **Input Layer:** `river_buffers_500m.gpkg`.

   * **Overlay Layer:** Selected district boundary polygon.

   * Save the output to your `outputs/` folder.

---

## 2. Zonal Statistics Laboratory
To calculate the average elevation per district:

1. Load `nepal_districts.gpkg` and your digital elevation raster (`basin_dem.tif`).

2. Go to the Processing Toolbox and search for **Zonal Statistics**.

3. **Input Layer (Raster):** `basin_dem.tif`.

4. **Vector Layer containing zones:** `nepal_districts.gpkg`.

5. **Output Column Prefix:** `elev_`.

6. Click **Statistics to calculate** and check `Mean`, `Max`, and `Min`.

7. Click **Run**.

8. Right-click `nepal_districts.gpkg` > **Open Attribute Table**. You will see three new attribute columns containing the calculated elevation statistics for each district polygon.
