# Advanced Cartography and Reporting

Maps are the final link between geospatial analysis and planning decisions. Advanced cartography ensures that maps are accurate, legible, and visually balanced.

---

## 1. Professional Print Compositions

Designing a professional map layout requires organizing key elements to guide the map reader's eye:

*   **Visual Balance:** Place the main map canvas prominently. Align title, legend, scale, and references cleanly.

*   **Scale Bar Configuration:**
    
    Scale bars must display logical, rounded numbers (e.g., $10\text{ km}$, $50\text{ km}$, rather than $13.78\text{ km}$).
    
    Configure proper step increments in the properties panel.

*   **Graticules (Coordinate Grid Overlays):**
    
    Include grid overlays with latitude/longitude lines (for degrees/minutes/seconds) or projected coordinates (for meters).
    
    Label grid coordinates on the outer border of the frame.

*   **Typography Hierarchy:**
    
    *   **Title:** Large bold font ($20-28\text{ pt}$), sans-serif (e.g., Arial or Montserrat) for readability.
    
    *   **Labels:** Intermediate font sizes. Use text buffers (halos) on multi-color layers to ensure text legibility.
    
    *   **Source/Metadata:** Small font ($8-9\text{ pt}$) at the bottom detailing the coordinate reference system, data creator, and acquisition dates.

*   **Overview Inset Map:**
    
    A small map showing the location of your main map within a broader regional context (e.g., showing a specific watershed highlighted inside the country boundary of Nepal).
    
    *QGIS Configuration:* Lock the layers and style configurations for the inset map canvas so it does not shift when you edit the main map canvas.

---

## 2. Dynamic Mapbook Generation (QGIS Atlas)

The **Atlas** tool in QGIS automates the creation of a series of maps. This is useful for generating standardized reports for multiple administrative units (e.g., districts, sub-basins).

*   **Coverage Layer:**
    
    A vector layer containing the boundaries that define each page.
    
    The atlas will iterate through each feature, centering and zooming the map canvas on that feature's boundary.

*   **Dynamic Labels:**
    
    Use QGIS expressions to dynamically update titles, dates, or page numbers based on the attribute table of the coverage layer:
    
    `"Basin Name: " || [% "basin_name" %]`

*   **Scale Controls:**
    
    Choose between keeping a fixed scale for all pages, or using a dynamic scale that adjusts to fit the feature boundary with a buffer margin (e.g., $10\%$ margin).

---

## 3. Step-by-Step Exercise: Constructing an Atlas

In this exercise, we will build an automated map booklet for three river sub-basins.

1.  **Open Print Layout:**
    
    With your styled layers loaded in QGIS, go to **Project** > **New Print Layout...**.
    
    Name it `Basin_Atlas`.

2.  **Add Map Canvas:**
    
    Use the **Add Map** tool to draw a rectangle covering the left side of the canvas.

3.  **Enable Atlas Generation:**
    
    In the right-hand panel, select the **Atlas** tab.
    
    Check the box to **Generate an atlas**.
    
    Select the **Coverage Layer** (e.g., `sub_basins.gpkg`).
    
    Set the Page Name attribute to `basin_name`.

4.  **Configure Map Properties:**
    
    Click on the map item in your layout.
    
    In the **Item Properties** panel, check the box **Controlled by Atlas**.
    
    Select **Margin around feature** and set it to `10 %`.

5.  **Add Dynamic Title Text:**
    
    Use the **Add Label** tool.
    
    Insert the following expression:
    
    `Watershed Report: [% "basin_name" %]`

6.  **Preview and Export:**
    
    Click the **Preview Atlas** button in the toolbar. Use the arrow buttons to cycle through each sub-basin page.
    
    Go to **Atlas** > **Export Atlas as PDF...** to export all pages as a single PDF map booklet.
