# Geospatial Decision Support Systems

A **Geospatial Decision Support System (GDSS)** combines spatial analysis, mathematical models, and policy rules to help decision-makers evaluate alternative scenarios.

---

## 1. Multi-Criteria Decision Analysis (MCDA)

MCDA is a framework used to find optimal locations by evaluating multiple criteria (e.g., land suitability, slope gradients, cost, proximity to markets, environmental regulations).

```text
  MCDA WORKFLOW PIPELINE
  +-------------+    +---------------+    +-----------------+
  | Raster/      | -> | Standardize   | -> | Apply Weights   | ---\
  | Vector Data |    | Range [0-100] |    | (AHP / Matrix)  |    \
  +-------------+    +---------------+    +-----------------+     \  +-----------------+
                                                                   +> | Weighted Linear | -> Best Sites
  +-------------+    +---------------+                            /   | Combination    |
  | Constraint  | -> | Binary Mask   | --------------------------/    +-----------------+
  | Layer       |    | [0 or 1]      |
  +-------------+    +---------------+
```

*   **Criterion Standardisation:**
    
    Raw values (e.g., slopes in degrees, distance in meters) cannot be added together directly.
    
    They must be standardized into a uniform scale (e.g., $0$ to $100$ or $0$ to $10$), where higher numbers indicate better suitability.

*   **Constraint Masking:**
    
    Constraints are binary exclusions (e.g., water bodies, protected national parks).
    
    Represented as $1$ (suitable to build) and $0$ (strictly excluded).

*   **Synthesis (Weighted Linear Combination):**
    
    The final suitability score is calculated by multiplying each standardized criterion score by its assigned weight, and then multiplying the total by the constraint mask.
    
    $$\text{Score} = \left( \sum_{i=1}^{n} w_i x_i \right) \times \prod_{j=1}^{m} c_j$$
    
    Where $w_i$ is criterion weight, $x_i$ is standardized score, and $c_j$ is the binary constraint.

---

## 2. Analytical Hierarchy Process (AHP)

AHP is a structured technique for organizing and analyzing complex decisions, developed by Thomas L. Saaty. It determines weights by using pairwise comparison matrices.

*   **Pairwise Comparison Matrix:**
    
    Criteria are compared in pairs on a scale of $1$ (equal importance) to $9$ (extremely more important).
    
    If Criterion A has an importance of $5$ relative to Criterion B, then Criterion B has an importance of $1/5$ ($0.2$) relative to Criterion A.

*   **Consistency Check:**
    
    To ensure comparison judgements are logical, we calculate the Consistency Ratio (CR):
    
    $$\text{CR} = \frac{\text{CI}}{\text{RI}}$$
    
    Where:
    
    $$\text{CI} = \frac{\lambda_{\text{max}} - n}{n - 1}$$
    
    *   $n$ is the number of criteria.
    
    *   $\lambda_{\text{max}}$ is the principal eigenvalue.
    
    *   $\text{RI}$ is the Random Index (standard value based on matrix size).
    
    *   **CR Threshold:** The CR must be $< 0.10$ ($10\%$) to be considered consistent. If CR $\ge 0.10$, pairwise rankings must be revised.

---

## 3. Hydropower Suitability Analysis Exercise

We will model suitable runoff hydropower project locations using three parameters: drainage network density (Weight $= 50\%$), slope gradient (Weight $= 30\%$), and distance to transmission lines (Weight $= 20\%$). Conservation areas represent a binary constraint.

1.  **Prepare Input Rasters:**
    
    *   `flow_accum_class.tif` (reclassified values $1-10$, where $10$ indicates high accumulation).
    
    *   `slope_class.tif` (reclassified values $1-10$, where $10$ represents optimal steep head gradient).
    
    *   `grid_proximity_class.tif` (reclassified values $1-10$, where $10$ is closest to the grid).
    
    *   `conservation_mask.tif` ($0$ inside national parks, $1$ everywhere else).

2.  **Enter Weighted Expression in Raster Calculator:**
    
    Input the equation:
    
    `(("flow_accum_class@1" * 0.50) + ("slope_class@1" * 0.30) + ("grid_proximity_class@1" * 0.20)) * "conservation_mask@1"`
    
    Save output as `hydropower_suitability.tif`.

3.  **Identify Candidate Locations:**
    
    Style the output with a singleband pseudocolor ramp.
    
    Extract pixels with suitability scores $> 8.0$ and convert them to vector points representing candidate intake locations.
