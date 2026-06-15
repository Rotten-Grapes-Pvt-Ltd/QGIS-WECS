# Layer Symbology and Styling

Symbology is the visual representation of geographic features. Proper styling is crucial for making maps legible and communicating analytical results.

---

## 1. Vector Symbology Types

* **Single Symbol:** Renders all features in the layer using the same style (e.g., all river lines drawn in the same shade of blue).

* **Categorized:** Styles features based on a nominal or qualitative attribute field (e.g., styling land use polygons based on their classification value: forest is green, urban is red, water is blue).

* **Graduated:** Styles features based on a continuous quantitative attribute field (e.g., styling sub-districts from light yellow to dark red based on population density).

---

## 2. Advanced Rule-Based Styling
Rule-based symbology uses expressions to define complex styling conditions:

* **Example:**
  ```text
  Rule 1: "Elevation" > 3000 AND "Slopes" > 25  --> Draw in Red (High Landslide Risk)
  Rule 2: "Elevation" <= 3000 OR "Slopes" <= 25  --> Draw in Green (Low Landslide Risk)
  ```

---

## 3. Dynamic Labels
Labels display text attributes directly on the map canvas.

* **Formatting:** You can set fonts, colors, and sizes.

* **Placement Rules:** Enforce rules to prevent text overlapping (e.g., labels along river lines should curve to match the path of the stream).

* **Halo / Buffer:** Adding a semi-transparent text buffer around labels to make them readable when drawn on top of complex backgrounds (like satellite images).
