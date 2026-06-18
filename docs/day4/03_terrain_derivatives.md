# Primary and Secondary Terrain Derivatives

Terrain derivatives are surface rasters calculated directly from Digital Elevation Models. They are divided into **Primary Derivatives** (representing geometry and orientation) and **Secondary Derivatives** (representing mathematical combinations of primary metrics to model ecological or hydrological processes). This section covers slope, aspect, hillshade, curvature, and the Topographic Wetness Index (TWI).

---

## 1. Primary Terrain Derivatives

Primary derivatives represent the direct geometric shape of the landscape at each grid cell, typically computed using neighborhood algorithms (inspecting a $3 \times 3$ pixel window).

### Slope (Topographic Gradient)

Measures the rate of change of elevation in the direction of steepest descent.

* **Mathematical Definition:** Slope is the first derivative of elevation. Calculated using the partial derivatives of elevation ($z$) in the horizontal ($x$) and vertical ($y$) directions:
  $$\text{Slope} = \arctan \left( \sqrt{\left(\frac{\partial z}{\partial x}\right)^2 + \left(\frac{\partial z}{\partial y}\right)^2} \right)$$

* **Measurement Units:**
  * **Degrees:** Ranges from $0^{\circ}$ (flat plain) to $90^{\circ}$ (vertical cliff).
  * **Percentage:** Calculated as rise over run ($(\text{Vertical Change} / \text{Horizontal Distance}) \times 100$). A $45^{\circ}$ slope equals $100\%$, while steeper cliffs approach infinity.

* **Hydrological Role:** Direct controller of overland flow velocity, kinetic energy of surface runoff, and soil erosion capacity (used in the USLE/RUSLE LS factor).

### Aspect (Slope Orientation)

The compass direction that the slope faces, measured clockwise from North ($0^{\circ}$ to $360^{\circ}$).

* **Hydrological Role:** Determines solar radiation exposure. In the Himalayas, South-facing slopes receive significantly more sunlight than North-facing slopes, resulting in faster snowmelt rates, higher soil evaporation, and distinct vegetation patterns.

### Hillshade (Shaded Relief)

A visual simulation representing the illumination of the surface under a virtual light source.

* **Configuration Parameters:**
  * **Azimuth:** The compass direction of the sun (standard default is $315^{\circ}$ - Northwest).
  * **Altitude:** The angle of the sun above the horizon (standard default is $45^{\circ}$).

* **Hydrological Role:** Used as a transparent backdrop layer overlaying DEMs to visualize ridges, canyons, and drainage routes.

---

## 2. Secondary Terrain Derivatives

Secondary derivatives combine primary parameters to model spatial processes like flow concentration or soil saturation.

### Planform and Profile Curvatures

Curvature is the rate of change of slope or aspect, representing the acceleration/deceleration and convergence/divergence of water across the terrain.

* **Profile Curvature (Slope-wise):** Calculated parallel to the direction of slope. Controls the acceleration or deceleration of surface runoff. A convex profile curvature accelerates water flow (increasing erosion), while a concave profile decelerates water flow (increasing sedimentation).

* **Planform Curvature (Contour-wise):** Calculated perpendicular to the direction of slope. Controls the convergence or divergence of runoff. Concave planform curvature forces flow to converge (creating channels in valleys), while convex planform curvature forces flow to diverge (spreading water away from ridges).

```text
    PROFILE CURVATURE                       PLANFORM CURVATURE
    (Flow Acceleration)                     (Flow Concentration)
    
    Convex (\ ) --> Accelerates Flow       Convex (/\) --> Diverges Flow
    Concave ( \_) --> Decelerates Flow      Concave (\/) --> Converges Flow
```

---

## 3. Topographic Wetness Index (TWI)

The **Topographic Wetness Index (TWI)** (or compound topographic index) is a steady-state wetness index used to predict zones of soil saturation and wetland occurrence.

* **Mathematical Formula:**
  $$\text{TWI} = \ln \left( \frac{\alpha}{\tan \beta} \right)$$
  Where:
  * $\alpha$ is the specific catchment area (the local contributing area per unit contour length, representing water supply).
  * $\beta$ is the local slope angle in degrees (representing the gravitational gradient driving water export).

* **Interpretation:**
  * High TWI values indicate flat valley bottoms with large upstream contributing areas (high potential for saturation and swamp formation).
  * Low TWI values indicate steep ridge areas with small drainage catchments (dry zones).

---

## 4. Calculating Derivatives in QGIS

QGIS Processing Toolbox includes tools to calculate these derivatives:

* **GDAL Slope/Aspect/Hillshade:** Go to **Raster** > **Analysis** > **Slope** (or **Aspect** / **Hillshade**). These run fast, native GDAL C++ routines.

* **SAGA Slope, Aspect, Curvature:** Select SAGA's **Slope, Aspect, Curvature** tool inside the Processing Toolbox for advanced curvature outputs.

* **SAGA TWI:** Locate **Topographic Wetness Index** under SAGA's terrain analysis algorithms. It requires a filled DEM and a calculated specific catchment area raster as inputs.
