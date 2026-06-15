# Introduction to Spatial Databases

Flat files (like shapefiles) are inefficient for managing large, multi-user enterprise datasets. Spatial databases resolve this issue by integrating GIS capabilities directly into database management systems (DBMS).

---

## 1. PostgreSQL and PostGIS

* **PostgreSQL:** An open-source object-relational database.

* **PostGIS:** An extension that adds support for geographic objects, spatial index grids (R-Tree), and spatial SQL query functions.

---

## 2. Spatial SQL Syntax
By using PostGIS, you can perform spatial operations directly using SQL queries:
```sql
-- Calculate the area of all catchments in square kilometers
SELECT name, ST_Area(geom) / 1000000 AS area_sqkm
FROM catchment_boundaries;
```
