# Attribute Data and Table Operations

Every spatial dataset combines geometric coordinate representations with tabular properties (attribute tables). Managing, querying, and updating these tables is a fundamental skill for geoprocessing workflows. This section details field configurations, data types, dynamic calculations using the **Field Calculator**, and relational database joins.

---

## 1. Relational Attribute Tables and Data Types

An attribute table is organized in a relational structure containing columns (**Fields**) and rows (**Records**):

```text
    +---------------------------------------------------------------------------------+
    | FID | NAME      | CONTINENT | POP_EST    | GDP_MD_EST | ISO_A3 | Last_Updated   |  <-- FIELDS (Columns)
    +---------------------------------------------------------------------------------+
    | 1   | Nepal     | Asia      | 29164578   | 29813.00   | NPL    | 2026-05-10     |  <-- RECORD (Row)
    | 2   | India     | Asia      | 1380004385 | 2875142.00 | IND    | 2026-05-12     |
    +---------------------------------------------------------------------------------+
```

When creating or editing fields, you must define the correct **Data Type**. Choosing the wrong type can cause calculation errors or data truncation:

* **Integer (Whole Number):** For counts, ranks, or identifiers without decimals (e.g., `FID`, population estimate `POP_EST`, scale rank `scalerank` in the rivers layer).

* **Real / Double (Decimal Number):** For continuous measurements, coordinates, and lengths (e.g., GDP estimate `GDP_MD_EST`, area calculations, coordinate values).

* **Text / String:** For alphanumeric labels, names, and codes (e.g., country name `NAME`, continent `CONTINENT`, country code `ISO_A3`).

* **Date / Date & Time:** For recording collection timestamps or metadata updates (`Last_Updated`).

* **Boolean:** Binary flag values (e.g., whether a country is landlocked).

---

## 2. The Field Calculator and Expressions

The **Field Calculator** allows you to create new fields or update existing attribute values using mathematical equations, string manipulations, or geometric properties.

* **Accessing:** Open the Attribute Table (`F6`), click the abacus icon (or press `Ctrl+I`), or click the Field Calculator icon in the Layer Styling Panel.

### Dynamic Geometry Fields

Unlike static attributes, geometry variables compute values dynamically based on the geographic shape of the feature. These functions are prefixed with a dollar sign (`$`):

* `$area`: Computes the polygon area in project units. To calculate and store country areas in square kilometers ($km^2$) for the `ne_110m_admin_0_countries` layer:
  `$area / 1000000`

* `$length`: Computes the length of line features. To calculate river segment lengths in kilometers for `ne_10m_rivers_lake_centerlines`:
  `$length / 1000`

* `$perimeter`: Computes the boundary length of polygons.

* `$x` and `$y`: Extracts the coordinate center points of points (e.g., coordinates of cities in `ne_10m_populated_places`).

> [!IMPORTANT]
> Geometry calculations depend heavily on the layer's Coordinate Reference System (CRS). If your layer is in a Geographic Coordinate System (GCS, such as WGS 84 / EPSG:4326), `$area` will be calculated in **square degrees** rather than square meters, resulting in unusable metrics. Always ensure your layers are projected (e.g., UTM Zone 45N / EPSG:32645) before performing geometry calculations.

### Attribute Calculations and Logic Expressions

You can write SQL-like code blocks to combine text columns or apply conditional categorization rules:

* **String Concatenation:** Combine country name and ISO code from `ne_110m_admin_0_countries` into a single label using the `||` operator:
  `"NAME" || ' (' || "ISO_A3" || ')'` (produces `'Nepal (NPL)'`)

* **Conditional Category Formatting (`CASE` statements):** Categorize countries based on population scale (`POP_EST`):
  ```sql
  CASE 
      WHEN "POP_EST" > 100000000 THEN 'Very Large Population'
      WHEN "POP_EST" > 10000000 AND "POP_EST" <= 100000000 THEN 'Large Population'
      WHEN "POP_EST" > 1000000 AND "POP_EST" <= 10000000 THEN 'Medium Population'
      ELSE 'Small Population'
  END
  ```

### Virtual Fields vs. Physical Fields

When creating a new field in the Field Calculator, you can check the box for **Create virtual field**:

* **Physical Fields:** Saved directly to the underlying data source (e.g., GeoPackage database or Shapefile). The values are calculated once and stored statically.
* **Virtual Fields:** Saved only within the QGIS project file (`.qgz`). The expression is evaluated dynamically in real-time every time you view the map or attribute table.
* **Application:** Ideal for area (`$area`) or length (`$length`) calculations. If you modify or edit a polygon's shape, a virtual field will instantly recalculate the new area, whereas a physical field will retain the old, outdated static value.

### Advanced Aggregate Expressions

QGIS allows you to run calculations across multiple records in a table using aggregate functions (which optionally accept group-by clauses).

* **Percentage of Regional Population:** Calculate what percentage of a continent's total population each country represents. Run the following expression on the `ne_110m_admin_0_countries` layer:
  ```sql
  "POP_EST" / sum("POP_EST", group_by:="CONTINENT") * 100
  ```
* **Deviation from Average GDP:** Calculate how much a country's GDP deviates from the global average:
  ```sql
  "GDP_MD_EST" - mean("GDP_MD_EST")
  ```

---

## 3. Tabular Relational Joins

A **Tabular Join** links an external, non-spatial table (like an Excel sheet, CSV file, or SQL database table containing country-level rainfall statistics) to a spatial layer using a shared common field (**Key**).

```text
    +--------------------------------------+                +-------------------------------+
    |  Spatial Countries Layer (Shapefile)  |                |      External CSV Table       |
    +--------------------------------------+                +-------------------------------+
    | NAME      | ISO_A3 (Key)             | -- JOIN ON --  | Country_Code  | Ann_Rain_mm   |
    +-----------+--------------------------+                +---------------+---------------+
    | Nepal     | NPL                      |                | NPL           | 1500.5 mm     |
    | India     | IND                      |                | IND           | 1080.2 mm     |
    +--------------------------------------+                +-------------------------------+
```

### Steps to Perform a Join in QGIS:

1. Import both the spatial layer `ne_110m_admin_0_countries` and the external CSV table containing meteorological records into QGIS.

2. Right-click the spatial layer and open **Properties** > **Joins**.

3. Click the green '+' icon at the bottom of the window.

4. **Join Layer:** Select the external CSV table.

5. **Join Field:** Select the common key in the CSV table (e.g., `Country_Code`).

6. **Target Field:** Select the matching key in the spatial layer (e.g., `ISO_A3`).

7. Under **Joined Fields**, check the boxes to only import the columns you need (e.g., `Ann_Rain_mm`).

8. Click **OK** and **Apply**. Open the spatial layer's attribute table to verify that the rainfall values are now appended to the spatial features.

> [!CAUTION]
> Key columns must have identical data types. If the key in your vector layer is defined as an **Integer** but the key in the CSV is imported as a **String/Text** (or vice-versa), QGIS will fail to match the records, resulting in empty joined fields. You can resolve this by writing a typecast expression (e.g., `to_int("Key")`) in the Field Calculator to create a matching field.

---

## 4. Spatial Joins (Join Attributes by Location)

While a tabular join links datasets using an alphanumeric key, a **Spatial Join** links attributes based on their geographic relationship (e.g., intersection, containment, or proximity).

```text
    SPATIAL JOIN CONCEPT (CONTAINMENT)
    +----------------------------------+
    | Polygon: Country (Name: Nepal)   |
    |  * Point: City (Kathmandu)       |  <-- City point inherits the attribute
    |                                  |      "Country Name: Nepal" by location
    +----------------------------------+
```

* **Application:** Transferring country boundary properties to point-based populated places to identify which country each city belongs to.

### Exercise: Spatial Join of Cities and Countries

1. Ensure both `ne_110m_admin_0_countries` (polygon) and `ne_10m_populated_places` (point) are loaded.
2. Go to the Processing Toolbox (`Ctrl+Alt+T`) and search for **Join attributes by location**.
3. Set the parameters:
   * **Input Layer (Target):** `ne_10m_populated_places` (the points we want to enrich).
   * **Join Layer (Source):** `ne_110m_admin_0_countries` (the polygon boundaries containing attributes).
   * **Geometric predicate:** Select **within** (points located within the polygons).
   * **Fields to add:** Click `...` and select `NAME` and `CONTINENT` (to prevent importing all country columns).
   * **Joined layer:** Click `...` and save to your GeoPackage database as `cities_joined_countries`.
4. Click **Run**. Open the attribute table of the output point layer. Verify that every city record now contains a field with the name and continent of its containing country.

---

## 5. Modifying Table Schema and Refactoring Fields

In GIS data management, you frequently need to modify the structure (schema) of a table—such as renaming fields, changing field types (e.g., text to integer), or deleting obsolete columns. Standard edits inside the attribute table do not allow renaming columns or altering their types to prevent database corruption.

To make these structural schema changes, use the **Refactor Fields** tool:

1. Open the **Processing Toolbox** and search for **Refactor Fields**.
2. **Input Layer:** Select the layer you want to modify (e.g., `ne_110m_admin_0_countries`).
3. In the mapping table, you can:
   * **Rename Fields:** Change the name in the **Output name** column (useful for converting long column names to fit Shapefile $10$-character limitations).
   * **Typecast Fields:** Change the data type in the **Type** column (e.g., converting a text-based numeric ID column to an `Integer64` or `Double`).
   * **Reorder Fields:** Use the arrow icons to move columns up or down in the schema.
   * **Delete Fields:** Click the red 'Delete' icon next to columns you wish to remove.
4. Set the output to save as a new layer or database table and click **Run**.

---

## 6. Summary Statistics and the Statistics Panel

You do not always need to write expressions or export tables to understand the statistical distribution of your attribute values. QGIS provides a built-in statistics panel to view summary metrics on-the-fly.

* **Accessing:** Go to **View** > **Panels** > **Statistics Panel** (or click the $\sum$ icon in the toolbar).

```text
    STATISTICS PANEL SUMMARY
    +----------------------------------+
    | Field: POP_EST                   |
    +----------------------------------+
    | Count  : 177                     |
    | Sum    : 7,562,354,120           |
    | Mean   : 42,725,164              |
    | StDev  : 149,204,112             |
    | Median : 9,842,500               |
    +----------------------------------+
```

### Viewing Attribute Distributions:
1. Open the **Statistics Panel**.
2. Select your layer (e.g., `ne_110m_admin_0_countries`) from the top dropdown menu.
3. Select the target attribute field (e.g., `POP_EST`) from the second dropdown menu.
4. The panel instantly displays descriptive statistics, including Count, Sum, Mean, Median, Standard Deviation, Minimum, Maximum, Range, and Deciles.
5. Check the box **Selected features only** to view statistics for a subset of features you have highlighted on your map canvas.

---

## 7. Selecting Features by Attributes

Instead of selecting features manually, you can use the **Select by Expression** tool (`Ctrl+F3`) to run logical query filters on your attribute table:

* Find all populated places with a population greater than 5 million:
  `"pop_max" > 5000000`

* Filter countries located in South Asia:
  `"ISO_A3" IN ('NPL', 'IND', 'BTN', 'BGD', 'LKA', 'PAK')`

* Combine scale rank and feature type for rivers:
  `"scalerank" <= 3 AND "featurecla" = 'River'`
