# Terrain Derivatives

Terrain derivatives are surfaces calculated from digital elevation datasets to analyze topographic gradients and solar exposure.

---

## 1. Primary Terrain Surfaces

* **Slope:** The rate of change of elevation, measured in degrees ($0^{\circ}$ to $90^{\circ}$) or percentage ($0\%$ to $\infty$). Slope determines water velocity, runoff acceleration, and soil erosion potential.

* **Aspect:** The compass direction that a slope faces ($0^{\circ}$ to $360^{\circ}$). Aspect determines solar exposure, which influences snowmelt timing and evaporation rates.

* **Hillshade:** A shaded relief visualization created by simulating the sun's position (altitude and azimuth) in the sky. This is used as a background layer to visualize terrain relief.

---

## 2. Advanced Curvature Derivatives

* **Profile Curvature:** Steepness changes along the direction of slope. Controls flow acceleration and deceleration.

* **Planform Curvature:** Curvature transverse to the slope direction. Controls flow convergence (accumulation in valleys) and divergence (spreading on ridges).
