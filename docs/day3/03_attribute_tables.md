# Attribute Data and Table Operations

Geospatial layers combine geometry with tabular attributes. Managing and querying these tables is a fundamental GIS skill.

---

## 1. The Field Calculator
The **Field Calculator** allows creating new attributes or updating existing fields using math equations, text formatting, and geometry calculations.

* **Accessing:** Open the attribute table and click the abacus icon (or `Ctrl+I`).

* **Dynamic Variables:**

  * `$area`: Calculates the area of polygon features (in project units, e.g., square meters).

  * `$length`: Calculates the length of line features.

  * `$id`: Assigns a sequential unique identifier to each row.

---

## 2. Tabular Joins
A join links an external table (such as an Excel spreadsheet of rainfall records) to a spatial layer (such as a rain gauge point shapefile) based on a shared common field (e.g., `Station_ID`).

```text
  [ Spatial Layer Table ]               [ External Excel Table ]
  FID | Station_Name | ID ----Join----> ID | Rainfall_mm | Date
   1  | Koshi West   | 401             401 | 124.5       | 2026-06-14
```
