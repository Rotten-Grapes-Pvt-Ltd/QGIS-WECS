# Flood Mapping and Water Resource Applications

Geospatial terrain analysis is crucial for modeling flood hazards, assessing community vulnerability, and planning infrastructure like reservoirs and check dams. This section covers GIS-hydraulic model coupling (HEC-RAS), the Height Above Nearest Drainage (HAND) inundation model, and stage-volume capacity analysis for reservoirs.

---

## 1. Hydraulic Coupling: QGIS and HEC-RAS

Hydraulic modeling simulates how water moves through a channel network. We couple GIS with hydraulic software (such as HEC-RAS) in a two-phase workflow:

### Pre-Processing (Terrain Extraction)

1. Use a high-resolution DEM to digitize the **Stream Centerline**, **Bank Lines**, and **Flow Path Centerlines** (left and right overbanks).

2. Draw **Cross-Section Cut Lines** perpendicular to the river flow path.

3. Extract elevation profiles along these cross-sections.

4. Export these geometries as a GIS-formatted XML file and import them into HEC-RAS to construct the 1D/2D channel geometry model.

### Post-Processing (Inundation Mapping)

1. Run the flood simulation in HEC-RAS using boundary discharge rates (e.g., a $100\text{-year}$ return storm).

2. Export the water surface elevation raster or polygon outputs from HEC-RAS.

3. Import the water surface layer back into QGIS.

4. **Flood Depth Mapping:** Subtract the bare-earth DTM elevation grid from the water surface elevation grid in the Raster Calculator:
   `"Water_Surface_Elevation@1" - "Terrain_DTM@1"`

5. Reclassify the output to display only positive values, showing water depths across flooded communities and agricultural land.

---

## 2. The HAND (Height Above Nearest Drainage) Model

**Height Above Nearest Drainage (HAND)** is a terrain model that normalizes topography based on its relative vertical height above the nearest stream channel cell it drains to.

```text
    HAND CONCEPT ELEVATION PROFILE
    
    Terrain cell [A] (Elevation = 100m, HAND = 15m)
           \
            \__ Terrain cell [B] (Elevation = 90m, HAND = 5m)
               \
                \__ Stream cell [C] (Elevation = 85m, HAND = 0m)
    ===============================================================
    [If water rises by 5m, cell B is flooded; cell A is safe]
```

### The HAND Workflow:

1. **Calculate Flow Direction:** Run the D8 algorithm to map the downslope direction paths.

2. **Delineate the Stream Grid:** Threshold the flow accumulation grid to define active river channels.

3. **Stream Elevation Raster:** Create a raster containing the elevation values of stream pixels, assigning NoData to land pixels.

4. **Flow Path Routing:** Route the stream elevations upslope along the D8 flow path direction vectors to populate every land cell with the elevation of its target stream cell.

5. **HAND Calculation:** Subtract this stream elevation raster from the original DEM.
   $$\text{HAND} = \text{Cell Elevation} - \text{Nearest Stream Elevation}$$

### Applications:

Unlike complex hydrodynamic models, HAND runs fast on raw DEMs. A HAND pixel value of $5\text{ m}$ means that if the local river level rises by $5\text{ meters}$, that cell will be flooded, making it an excellent tool for rapid flood hazard mapping.

---

## 3. Reservoir Capacity Stage-Volume Analysis

When planning a hydropower or irrigation dam, engineers must determine the reservoir's capacity (volume) and surface area at varying water heights (stage).

```text
    RESERVOIR STAGE-VOLUME CURVE
    Stage (Height in meters)
      |         /  Surface Area (m2)
      |        /
      |       /   /  Storage Volume (m3)
      |      /   /
      |     /   /
      +----+---+-------
           Volume / Area
```

### Calculation Steps in QGIS:

1. Delineate the catchment boundary upstream of the planned dam axis.

2. Identify the planned water elevation levels (stages), e.g., from a minimum pool level of $450\text{ m}$ to a maximum flood level of $500\text{ m}$.

3. Search for the **Raster Surface Volume** tool in the Processing Toolbox.

4. **Input Layer:** Select the filled DEM.

5. **Base Threshold:** Enter the target water level height (e.g., `480`).

6. **Method:** Select **Count Only Below** (to calculate the volume of the depression beneath the water surface).

7. Click **Run**. QGIS will output:
   * **Area ($m^2$):** The surface area of the reservoir at that pool height.
   * **Volume ($m^3$):** The total storage capacity.

8. Repeat this calculation at $5\text{-meter}$ height increments. Plot the results in Excel to construct **Stage-Area-Volume Curves** to size the dam's spillway and active storage parameters.
