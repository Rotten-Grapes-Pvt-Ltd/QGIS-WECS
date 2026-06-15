# Workflow Automation and Model Builder

Performing spatial analysis manually is repetitive and prone to errors. Automation allows building repeatable workflows that can be run on new datasets.

---

## 1. QGIS Graphical Model Builder
The **Graphical Model Builder** allows creating spatial models by connecting inputs, processing tools, and outputs in a visual flowchart.

```text
  [ Input DEM ] ------> [ Fill Sinks ] ------> [ Slope Tool ] ------> [ Output Slope ]
```

* **Workflow:**

  1. Define model inputs (e.g., a raster DEM layer and a watershed polygon mask).

  2. Drag processing algorithms from the toolbox into the workspace.

  3. Link the output of one tool as the input of the next (e.g., linking the filled DEM output to the Slope calculation input).

  4. Run the model as a single tool.
