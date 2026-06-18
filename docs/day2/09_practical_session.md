# Practical Session: Satellite Data Acquisition and Metadata Registry

This practical session guides you through searching, filtering, visualizing, and downloading satellite data using web portals (Copernicus CDSE Browser, USGS EarthExplorer), programmatic STAC APIs, and ObservEarth. You will establish your workspace and create a metadata registry for a selected river basin.

---

## 1. Setting Up the Project Workspace

Before downloading any datasets, organize your local project directories:

1. In your system's file manager or terminal, create the following subdirectories inside your main training folder:

   * `data/raw/sentinel2/` (For raw spectral bands)

   * `data/raw/dem/` (For digital elevation model grids)

   * `data/metadata/` (For metadata registry spreadsheets)

2. Keep these folders clean and dedicated strictly to raw, uncompressed inputs.

---

## 2. Searching and Filtering in Copernicus CDSE

The **Copernicus Data Space Ecosystem (CDSE)** is the primary web portal for accessing Sentinel imagery.

1. Open a web browser and navigate to [dataspace.copernicus.eu](https://dataspace.copernicus.eu/). Create a free account or log in.

2. Open the **CDSE Browser** map interface.

3. Search for your study area:

      * In the search bar, type a catchment name in Nepal (e.g., *"Bagmati River"* or *"Melamchi Basin"*) or draw a bounding box using the polygon drawing tool.

4. Set search criteria:

      * **Data Collection:** Sentinel-2 L2A (Bottom-Of-Atmosphere reflectance).

      * **Date Range:** Select a dry-season window (e.g., October to March) to minimize cloud interference.

      * **Cloud Cover Filter:** Slide the threshold to $< 10\%$.

5. Click **Search**. The system will return a list of matching satellite scenes.

---

## 3. Interactive Web-Based Visualizations

Before downloading files, inspect the bands online:

1. In the search results panel, select a low-cloud-cover scene over your study area.

2. Explore the visualization modes:

   * **True Color (B4, B3, B2):** Displays the scene in natural colors (similar to human vision). Inspect the sediment concentration in rivers (appearing brown/cyan).

   * **False Color (B8, B4, B3):** Displays the Near-Infrared (NIR) channel. Healthy vegetation will appear bright red, while water bodies will appear black or deep blue.

3. Note down the **Product Name** (unique ID) of your chosen scene (e.g., `S2B_MSIL2A_20241115T...`). You will need this for your registry.

---

## 4. Programmatic Searching via SpatioTemporal Asset Catalog (STAC)

Instead of searching manually, you can query satellite catalogs programmatically using Python.

1. Open your terminal or a Python development environment.

2. Install the standard STAC client library:
   ```bash
   pip install pystac-client pystac
   ```

3. Run a basic query block to discover Sentinel-2 scenes over a bounding box in Nepal (e.g., Kathmandu region):
   ```python
   from pystac_client import Client

   # Connect to the open-access Element84 STAC endpoint
   client = Client.open("https://earth-search.aws.element84.com/v1")

   # Define search criteria (Kathmandu bounding box, dry season, low clouds)
   search = client.search(
       max_items=5,
       collections=["sentinel-2-l2a"],
       bbox=[85.2, 27.6, 85.4, 27.8],
       datetime="2024-11-01/2024-12-31",
       query={"eo:cloud_cover": {"lt": 10}}
   )

   # Print matching scenes and their asset links
   for item in search.get_all_items():
       print(f"ID: {item.id} | Clouds: {item.properties['eo:cloud_cover']:.2f}%")
       # Print direct URLs to Green and NIR bands
       print(f"  Green Band: {item.assets['green'].href}")
       print(f"  NIR Band: {item.assets['nir'].href}")
   ```

---

## 5. Streaming and Downloading from ObservEarth

To bypass bulky raw zip files, utilize **ObservEarth** to download pre-clipped, analysis-ready datasets:

1. Open your web browser and navigate to [observearth.com](https://observearth.com).

2. Set up your user credentials or log in to the developer console.

3. Define your target basin boundary:

   * Draw a polygon over your selected watershed or upload your basin outline.

4. Query the API/Console:

   * Choose Sentinel-2 L2A as the base product.

   * Request automated extraction of NDWI (Normalized Difference Water Index) and NDVI bands.

5. Direct Download:

   * Download the clipped Green (Band 3) and NIR (Band 8) GeoTIFF files directly to your `data/raw/sentinel2/` directory.

---

## 6. Downloading Digital Elevation Models (DEMs)

Next, acquire topographic data using **USGS EarthExplorer**:

1. Navigate to [earthexplorer.usgs.gov](https://earthexplorer.usgs.gov/).

2. Define your spatial footprint using the map or coordinates.

3. Under **Data Sets**, expand **Digital Elevation** and select **SRTM** > **SRTM 1 Arc-Second Global**.

4. Search and download the DEM tiles covering your watershed in GeoTIFF format. Save them to `data/raw/dem/`.
