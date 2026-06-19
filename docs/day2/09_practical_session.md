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

---

## 7. Guided Mini-Assignment: Satellite Basin Data Acquisition and Metadata Registry

In this assignment, you will apply the search, filtering, and data discovery skills learned during Day 2 to download multispectral bands and digital elevation datasets for a target watershed in Nepal and construct a structured metadata catalog registry.

### Objective
Download and compile a structured, analysis-ready remote sensing dataset for a selected river basin in Nepal (e.g., Kosi, Gandaki, Karnali, or local tributaries like Bagmati, West Rapti, Babai), and build a metadata spreadsheet registering the critical satellite characteristics of the acquired assets.

### Selection of Target Basin
1. Select a river basin or sub-catchment in Nepal of interest to your work at WECS.

2. Identify its approximate geographic bounding box (coordinates in Decimal Degrees) or shape outline.

### Step-by-Step Requirements

#### Step 1: Sentinel-2 Multispectral Scene Search
Open Copernicus CDSE Browser or run a Python STAC API query:

1. Search for Sentinel-2 L2A imagery covering your target basin.

2. Set the search filter to a cloud cover threshold of $< 10\%$.

3. Find a dry-season scene (between November 2023 and April 2024) to ensure minimal snow cover and cloud interference.

4. Note down the unique scene **Product Identifier** (e.g., `S2A_MSIL2A_20231225...`).

#### Step 2: Digital Elevation Model (DEM) Search
Open USGS EarthExplorer or CDSE Browser:

1. Search for digital elevation grids (SRTM 1 Arc-Second or Copernicus 30m DEM) covering your target basin coordinates.

2. Note down the individual tile names/identifiers covering the basin.

#### Step 3: Downloading the Datasets
Download the following files to your structured local workspace:

1. For Sentinel-2, download the raw Green Band (Band 3) and NIR Band (Band 8) GeoTIFFs (either as individual bands via ObservEarth/STAC, or the full unzipped granule folder).

2. For the DEM, download the target tile GeoTIFFs.

3. Organize these files under:
   * `data/raw/sentinel2/`
   * `data/raw/dem/`

#### Step 4: Compiling the Metadata Registry
Create a CSV or Excel spreadsheet named `basin_metadata_registry.csv` inside `data/metadata/`. The spreadsheet must contain a table with the following column headers and completed information:

| Dataset | Sensor/Platform | Date of Acquisition | Spatial Resolution (m) | Cloud Cover % | Coordinate Reference System (CRS) | File Size (MB) | Scene/Tile Identifier |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Sentinel-2 Green Band** | Sentinel-2A / MSI | `[YYYY-MM-DD]` | $10\text{ m}$ | `[Value]` | `[e.g., EPSG:32645]` | `[Value]` | `[Full Scene ID]` |
| **Sentinel-2 NIR Band** | Sentinel-2A / MSI | `[YYYY-MM-DD]` | $10\text{ m}$ | `[Value]` | `[e.g., EPSG:32645]` | `[Value]` | `[Full Scene ID]` |
| **Elevation Grid (DEM)** | Shuttle Radar / SRTM | `[February 2000]` | $30\text{ m}$ | $0\%$ | `[e.g., EPSG:4326]` | `[Value]` | `[Tile Name]` |

### Submission Instructions
Compile your assignment outputs into a single ZIP archive named `Day2_Assignment_[YourName].zip` containing:

1. The compiled metadata registry spreadsheet (`data/metadata/basin_metadata_registry.csv` or `.xlsx`).

2. A screenshot of your structured local workspace folder tree showing the downloaded raw Sentinel-2 and DEM datasets in their respective directories.

3. A text file containing a brief paragraph explaining why the selected Sentinel-2 date and cloud cover thresholds are suitable for dry-season baseflow or water body mapping in your target basin.
