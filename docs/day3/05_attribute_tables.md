# Attribute Data and Table Operations

Every spatial dataset combines geometric coordinate representations with tabular properties (attribute tables). Managing, querying, and updating these tables is a fundamental skill for geoprocessing workflows. This section details field configurations, data types, dynamic calculations using the **Field Calculator**, and relational database joins.

---

## 1. Relational Attribute Tables and Data Types

An attribute table is organized in a relational structure containing columns (**Fields**) and rows (**Records**):

```text
    +-------------------------------------------------------------+
    | FID | Gauge_Name   | Elevation_m | Active | Install_Date    |  <-- FIELDS (Columns)
    +-------------------------------------------------------------+
    | 1   | Koshi Base   | 240.5       | True   | 2020-04-12      |  <-- RECORD (Row)
    | 2   | Karnali Low  | 185.2       | False  | 2018-09-15      |
    +-------------------------------------------------------------+
```

When creating or editing fields, you must define the correct **Data Type**. Choosing the wrong type can cause calculation errors or data truncation:

* **Integer (Whole Number):** For counts, ranks, or identifiers without decimals (e.g., `FID`, `scalerank`, population counts).

* **Real / Double (Decimal Number):** For continuous measurements, coordinates, and lengths (e.g., rainfall in mm, basin area in $km^2$, average temperatures).

* **Text / String:** For alphanumeric labels, names, and descriptions (e.g., station names, river classifications).

* **Date / Date & Time:** For recording collection timestamps, vital for multi-temporal climate analyses.

* **Boolean:** Binary flag values (e.g., `True` or `False`, `Active` or `Inactive`).

---

## 2. The Field Calculator and Expressions

The **Field Calculator** allows you to create new fields or update existing attribute values using mathematical equations, string manipulations, or geometric properties.

* **Accessing:** Open the Attribute Table (`F6`), click the abacus icon (or press `Ctrl+I`), or click the Field Calculator icon in the Layer Styling Panel.

### Dynamic Geometry Fields

Unlike static attributes, geometry variables compute values dynamically based on the geographic shape of the feature. These functions are prefixed with a dollar sign (`$`):

* `$area`: Computes the polygon area in project units (e.g., square meters if using a projected coordinate system). To convert the output to square kilometers ($km^2$), write the expression: 
  `$area / 1000000`

* `$length`: Computes the length of line features (e.g., river lengths). To convert meters to kilometers, write:
  `$length / 1000`

* `$perimeter`: Computes the boundary length of polygons.

* `$x` and `$y`: Extracts the coordinate center points of points or polygon centroids.

> [!IMPORTANT]
> Geometry calculations depend heavily on the layer's Coordinate Reference System (CRS). If your layer is in a Geographic Coordinate System (GCS, such as WGS 84 / EPSG:4326), `$area` will be calculated in **square degrees** rather than square meters, resulting in unusable metrics. Always ensure your layers are projected (e.g., UTM EPSG:32645) before performing geometry calculations.

### Attribute Calculations and Logic Expressions

You can write SQL-like code blocks to combine text columns or apply conditional categorization rules:

* **String Concatenation:** Combine text from multiple columns using the `||` operator:
  `"District_Name" || ' - Station: ' || "Gauge_Name"`

* **Conditional Category Formatting (`CASE` statements):**
  ```sql
  CASE 
      WHEN "Rainfall_Annual" > 2500 THEN 'Very High Rainfall'
      WHEN "Rainfall_Annual" > 1500 AND "Rainfall_Annual" <= 2500 THEN 'High Rainfall'
      WHEN "Rainfall_Annual" > 800 AND "Rainfall_Annual" <= 1500 THEN 'Moderate Rainfall'
      ELSE 'Low Rainfall'
  END
  ```

---

## 3. Tabular Relational Joins

A **Tabular Join** links an external, non-spatial table (like an Excel sheet, CSV file, or SQL database table containing rainfall logs) to a spatial layer (like a rain gauge point layer) using a shared common field (**Key**).

```text
    +--------------------------+                +-------------------------+
    |   Spatial Points Layer   |                |   External CSV Table    |
    +--------------------------+                +-------------------------+
    | Station_Name | Key_ID    | -- JOIN ON --  | Key_ID | Rainfall_2024  |
    +--------------+-----------+                +--------+----------------+
    | Kosi Base    | 401       |                | 401    | 1245.8 mm      |
    | Karnali Low  | 402       |                | 402    | 980.2 mm       |
    +--------------------------+                +-------------------------+
```

### Steps to Perform a Join in QGIS:

1. Import both the spatial layer and the external CSV table into QGIS.

2. Right-click the spatial layer and open **Properties** > **Joins**.

3. Click the green '+' icon at the bottom of the window.

4. **Join Layer:** Select the external CSV table.

5. **Join Field:** Select the common key in the CSV table (e.g., `StationID`).

6. **Target Field:** Select the matching key in the spatial layer (e.g., `gauge_id`).

7. Under **Joined Fields**, check the boxes to only import the columns you need (e.g., `Rainfall_2024`).

8. Click **OK** and **Apply**. Open the spatial layer's attribute table to verify that the rainfall values are now appended to the spatial features.

> [!CAUTION]
> Key columns must have identical data types. If the key in your vector layer is defined as an **Integer** (e.g., `401`) but the key in the CSV is imported as a **String/Text** (e.g., `'401'`), QGIS will fail to match the records, resulting in empty joined fields. You can resolve this by writing a typecast expression (e.g., `to_int("Key")`) in the Field Calculator to create a matching field.

---

## 4. Selecting Features by Attributes

Instead of selecting features manually, you can use the **Select by Expression** tool (`Ctrl+F3`) to run logical query filters on your attribute table:

* Find all stations above a specific elevation:
  `"Elevation_m" > 1500`

* Filter specific districts:
  `"District" IN ('Sunsari', 'Morang', 'Jhapa')`

* Combine spatial parameters and labels:
  `"Basin_Area" > 500 AND "Runoff_Coef" >= 0.6`
