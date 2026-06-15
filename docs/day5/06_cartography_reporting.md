# Advanced Cartography and Reporting

Maps are the primary communication link between technical analysts and policymakers. Advanced cartography ensures that maps are clear, professional, and accurate.

---

## 1. QGIS Layout Manager
The **Layout Manager** provides a dedicated workspace to compile map sheets:

* **Compositions:** Configuring paper sizes ($A4$, $A3$, $A0$), orientations (Landscape, Portrait), and margins.

* **Map Grid Overlays (Graticules):** Adding coordinate ticks along map borders to allow users to verify coordinates.

* **Scale Bars and Legend Formatting:** Formatting legends to show clear category descriptions (e.g., using "High Risk" instead of raw database codes like `Class_3_value`).

---

## 2. Layout Atlas Generation
The **Atlas** tool in QGIS allows automatically generating a booklet of maps. By defining a "coverage layer" (such as district boundaries), the layout manager will export a separate page for each district, centering the map canvas on that district's geometry automatically.
