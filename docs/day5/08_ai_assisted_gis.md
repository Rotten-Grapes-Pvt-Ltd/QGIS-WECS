# AI-Assisted GIS Workflows

Modern generative AI assistants help GIS professionals automate batch tasks, formulate complex query expressions, and write and troubleshoot scripts.

---

## 1. PyQGIS Scripting and Batch Processing

The Python API in QGIS (**PyQGIS**) allows running any processing tool programmatically and managing project layers.

*   **Accessing the Python Console:**
    
    Open QGIS. Go to **Plugins** > **Python Console** to access the interactive shell and editor.

*   **Basic Python Script Structure:**
    
    The script below loops through a folder of raster elevation files and clips each to a study boundary shapefile:

```python
import os
from qgis.core import QgsProject
import processing

# Define directories
input_folder = "/Users/krishnaglodha/Documents/work/wb/QGIS-WECS/data/raw_rasters/"
mask_layer = "/Users/krishnaglodha/Documents/work/wb/QGIS-WECS/data/boundary.gpkg"
output_folder = "/Users/krishnaglodha/Documents/work/wb/QGIS-WECS/data/clipped/"

# Create output folder if it doesn't exist
os.makedirs(output_folder, exist_ok=True)

# Loop and run GDAL clip
for filename in os.listdir(input_folder):
    if filename.endswith(".tif"):
        input_path = os.path.join(input_folder, filename)
        output_path = os.path.join(output_folder, "clipped_" + filename)
        
        params = {
            'INPUT': input_path,
            'MASK': mask_layer,
            'NODATA': -9999,
            'OUTPUT': output_path
        }
        
        processing.run("gdal:cliprasterbymasklayer", params)
        print(f"Successfully clipped: {filename}")
```

---

## 2. Prompting for Spatial SQL and GIS Expressions

When asking an AI assistant to generate expressions or queries, you must provide detailed context (schema names, field types, and target goals).

*   **Weak Prompt:**
    
    *"Write a SQL query to intersect rivers and buffers."*

*   **Effective AI Prompt:**
    
    *"I am working in a PostGIS spatial database. I have a vector table of rivers (`public.rivers_line`) with geometry column `geom` and attribute field `scalerank` (integer). I also have a district boundaries table (`public.districts_poly`) with geometry `geom` and name attribute `district_name`. Write a spatial SQL query to find the total length of major rivers (scalerank <= 3) that flow inside the district named 'Kathmandu'."*

*   **AI-Generated PostGIS SQL Output:**
    
    ```sql
    SELECT 
        d.district_name,
        SUM(ST_Length(ST_Intersection(r.geom, d.geom))) / 1000.0 AS river_length_km
    FROM 
        public.rivers_line AS r,
        public.districts_poly AS d
    WHERE 
        d.district_name = 'Kathmandu'
        AND r.scalerank <= 3
        AND ST_Intersects(r.geom, d.geom)
    GROUP BY 
        d.district_name;
    ```

---

## 3. Troubleshooting Common GIS Programming Errors

AI assistants can identify and resolve common bugs that occur during automation.

*   **Projection Mismatch Errors:**
    
    *Error symptoms:* Spatial queries return $0$ features or incorrect distances because layers are in different CRS (e.g. comparing WGS84 coordinates to UTM meters).
    
    *Solution:* Reproject layers to a common CRS (using `ST_Transform` in SQL or `QgsCoordinateTransform` in Python) before running overlay functions.

*   **Invalid Geometries:**
    
    *Error symptoms:* Buffer or Union operations fail with self-intersection errors.
    
    *Solution:* Run the QGIS **Fix Geometries** tool, or run `ST_MakeValid(geom)` in PostGIS.

*   **File Lock Issues:**
    
    *Error symptoms:* Python throws `PermissionError` when trying to overwrite a shapefile.
    
    *Solution:* QGIS locks active project layers. Remove the layer from the QGIS active canvas (`QgsProject.instance().removeMapLayer(layer)`) before overwriting it.
