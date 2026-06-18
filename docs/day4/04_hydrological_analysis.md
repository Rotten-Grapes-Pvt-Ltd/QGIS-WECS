# Hydrological Analysis and Flow Routing

Hydrological analysis models the movement of water across topographic surfaces. By analyzing coordinate grids, we can calculate flow directions, trace flow pathways, compute contributing drainage areas, and extract stream networks. This section covers flow direction algorithms, flow accumulation calculations, thresholding, and outlet selections.

---

## 1. Flow Direction Models

**Flow Direction** defines the downhill path that surface runoff will take from a target cell to its adjacent neighbors. QGIS utilizes two main algorithm families:

```text
    D8 FLOW ROUTING DIRECTION CODES
    [ 32 ][ 64 ][ 128 ]
    [ 16 ][CELL][  1  ]  <-- Water flows to only ONE neighbor (steepest path)
    [  8 ][  4 ][  2  ]
```

### D8 (Deterministic 8) Single Flow Direction

Assumes that water from a cell flows to only one of its eight neighboring cells (cardinal and diagonal) in the direction of the steepest downward gradient.

* **Direction Codes:** Direction is stored as power-of-two codes (e.g., $1 = \text{East}$, $2 = \text{Southeast}$, $4 = \text{South}$, $8 = \text{Southwest}$, etc.).

* **Pros/Cons:** Highly efficient and standard for extracting linear stream networks in high-relief mountainous catchments. However, it cannot model flow dispersion on divergent hillsides or flat plains, often creating artificial, parallel channel lines.

### Multiple Flow Direction (MFD)

Distributes downslope flow across all neighboring cells that are lower in elevation than the target cell. The flow fraction allocated to each neighbor is proportional to the slope gradient:

$$f_i = \frac{(\text{Slope}_i)^p}{\sum (\text{Slope}_j)^p}$$

Where $p$ is a weighting exponent.

* **Pros/Cons:** More realistic for modeling soil moisture distribution, solute transport, and overland sheet flow on hillsides, but unsuitable for extracting clean, single-line river channels.

---

## 2. Flow Accumulation Grids

The **Flow Accumulation** grid calculates the cumulative number of upstream cells (or total contributing area) draining into each grid cell.

$$\text{Flow Accumulation} = \sum \text{Upstream Cells} + 1$$

```text
    FLOW DIRECTIONS                 FLOW ACCUMULATION VALUES
    [ > ] [ v ] [ < ]               [ 0 ] [ 0 ] [ 0 ]  <-- Ridge cells
    [   ] [ v ] [   ]  --------->   [ 0 ] [ 2 ] [ 0 ]  
    [   ] [ v ] [   ]               [ 0 ] [ 5 ] [ 0 ]  <-- Channel stream line
```

* **No-Weight Accumulation:** Each cell contributes a value of $1$. The accumulation value represents the number of pixels draining through that cell. Multiply this by the pixel area (e.g., $900\text{ m}^2$ for $30\text{ m}$ grids) to calculate the contributing basin area in square meters.

* **Weighted Accumulation:** Each cell is multiplied by a secondary raster value (e.g., a spatial rainfall grid or soil runoff coefficient map). The output represents actual runoff volumes ($m^3/\text{sec}$) rather than simple geometric cell counts.

---

## 3. Thresholding and Stream Network Extraction

To extract river networks from the flow accumulation grid, you must define an accumulation **threshold** (minimum contributing area).

```text
    HIGH THRESHOLD (e.g. 5000 cells)       LOW THRESHOLD (e.g. 500 cells)
            |                                      \   |   /
            |  <-- Coarse network                   \  |  /  <-- Fine network
            |      (Major rivers only)               \ | /      (Includes gullies)
```

* **Coarse Threshold (High cell count):** Extracts only major river channels with large upstream basins (e.g., threshold set to $5000\text{ pixels}$). Useful for regional basin planning.

* **Fine Threshold (Low cell count):** Extracts minor drainage gullies, headwater creeks, and hillslope depressions (e.g., threshold set to $500\text{ pixels}$). Crucial for localized soil erosion modeling.

---

## 4. Pour Points and Basin Outlets

A **Pour Point** (or basin outlet) is the coordinate cell representing the exit point of a drainage network.

* **Placement:** Typically positioned at stream confluences, gauging stations, planned dam sites, or where a river crosses administrative boundaries.

* **The Snipping Workflow:** Because DEM cells may have sub-pixel offsets, the pour point must be snapped exactly to the pixel containing the highest flow accumulation value along the stream channel. Placing a pour point even one pixel offset from the flow accumulation line will result in a catchment size of near-zero.
