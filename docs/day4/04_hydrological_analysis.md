# Hydrological Analysis

Hydrological analysis uses conditioned elevation datasets to model flow routing, identify channels, and delineate drainage basins.

---

## 1. Flow Direction Algorithms
Flow direction defines the downhill path of water from cell to cell.

* **D8 (Deterministic 8) Algorithm:** Assumes water flows from a cell to only one of its eight neighboring cells in the direction of the steepest downward slope.

* **Multiple Flow Direction (MFD) Algorithms:** Distribute flow across multiple downslope neighbors, making them better suited for modeling flow dispersion in flat plains.

---

## 2. Flow Accumulation Grid
Flow accumulation calculates the cumulative number of upstream cells that drain into each cell in the grid:

$$\text{Flow Accumulation} = \sum \text{Upstream Cells}$$

```text
    +---+---+---+
    | 1 | 1 | 1 |  <-- Ridge cells (Accumulation = 0)
    +---+---+---+
    | 1 | 3 | 1 |  <-- Moderate accumulation
    +---+---+---+
    | 1 | 8 | 1 |  <-- Stream channel (High accumulation)
    +---+---+---+
```

---

## 3. Watershed Delineation
By setting a threshold value on the flow accumulation grid, the GIS can isolate pixels that behave as stream channels. The **Watershed Delineation** tool then traces flow directions upstream from a user-defined outlet point (pour point) to define the boundary of the basin.
