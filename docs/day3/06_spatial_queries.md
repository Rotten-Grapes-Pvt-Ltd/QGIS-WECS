# Spatial Queries and Analysis

Spatial queries select or aggregate features based on their spatial relationships (overlap, proximity, containment) rather than tabular attributes.

---

## 1. Zonal Statistics
**Zonal Statistics** calculates statistical summary values (mean, max, min, standard deviation) of a raster layer within the boundaries defined by a vector polygon layer.

```text
  [ Raster Grid (Rainfall) ]        [ Polygon Boundary (District) ]
  +---+---+---+
  | 10| 12| 15|                     /---------------\
  +---+---+---+  ------Overlay-----> |  Average: 13  |
  |  9| 13| 17|                     \---------------/
  +---+---+---+
```

* **Hydrological Application:** Overlaying a daily rainfall raster grid on top of sub-catchment boundaries to calculate the average rainfall volume entering each basin.

---

## 2. Nearest Neighbor Analysis
Finds the closest feature in one layer to features in another. For example, finding the nearest water quality monitoring station to a planned intake pipe.
