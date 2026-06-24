# Workflow Automation and Model Builder

Manual geospatial workflows are repetitive and vulnerable to user error. Automating these steps ensures that workflows are reproducible and can be shared or rerun on updated datasets.

---

## 1. The Graphical Model Builder

The **Graphical Model Builder** in QGIS allows analysts to construct geoprocessing pipelines by connecting inputs, processing tools, and outputs visually.

*   **Model Inputs:**
    
    Inputs can include raster layers, vector layers, map layers, numeric parameters, Boolean toggles, or folders.

*   **Geoprocessing Algorithms:**
    
    Any tool from the processing toolbox can be dragged into the model canvas.
    
    Output results from one tool can be configured as input parameters for downstream tools.

*   **Intermediate (Temporary) Outputs:**
    
    To avoid filling up hard drives with temporary vector/raster files, intermediate steps are written to RAM as temporary layers.
    
    Only the final output layer is saved to disk.

*   **Sharing Models:**
    
    Models are saved as `.model3` XML files.
    
    They can be stored in the QGIS profile directory, appearing as standard tools in the Processing Toolbox for other users.

---

## 2. Exercise: Delineating Steeps and Slopes from raw DEM

We will construct a graphical model that takes a raw DEM, performs sink filling, computes the slope, reclassifies the output, and exports the final raster.

```text
  MODEL BUILDER DESIGN CANVAS
  [ Input: DEM Raster ]
           |
           v
    (WhiteboxTools: Fill Depressions) -> [ Filled DEM (Intermediate) ]
                                 |
                                 v
                          (GDAL: Slope) -> [ Slope Raster (Intermediate) ]
                                                   |
                                                   v
                                          (Native: Reclassify) -> [ Output: High Slopes ]
```

1.  **Open Model Builder:**
    
    Go to **Processing** > **Graphical Modeler...** to open the canvas.

2.  **Define Inputs:**
    
    On the left panel, click the **Inputs** tab.
    
    Double-click **Raster Layer**.
    
    Set Parameter Name to `Raw DEM` and check **Mandatory**. Click **OK**.

3.  **Add Fill Sinks Tool:**
    
    Click the **Algorithms** tab.
    
    Search for **Fill Depressions (WhiteboxTools)**. Drag it onto the canvas.
    
    Set the input DEM parameter to use **Model Input** > `Raw DEM`.
    
    Leave output parameters blank (so they remain intermediate temporary files). Click **OK**.

4.  **Add Slope Tool:**
    
    Search for the **Slope (GDAL)** algorithm. Drag it onto the canvas.
    
    Configure the Input Layer to use the **Algorithm Output** > `Filled DEM` from the Fill Sinks step.
    
    Click **OK**.

5.  **Add Reclassify by Table Tool:**
    
    Search for **Reclassify by Table** (Native QGIS tool) and drag it.
    
    Set Input Raster to **Algorithm Output** > `Slope` from the GDAL step.
    
    Click on the Reclassification Table button and define the ranges:
    
    *   $0$ to $15$: Value $1$ (Gentle)
    
    *   $15$ to $30$: Value $2$ (Moderate)
    
    *   $30$ to $90$: Value $3$ (Steep)
    
    Under the output layer field, type `Reclassified Slope` (making this the model output). Click **OK**.

6.  **Run and Save:**
    
    Save the model as `Slope_Exclusion_Model` in your project folder.
    
    Click the green **Run** arrow at the top of the window. Select your raw input DEM file and click **Run**.
    
    The final reclassified slope layer will load into your layers panel.
