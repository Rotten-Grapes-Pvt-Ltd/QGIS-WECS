# Coordinate Reference Systems and Projections

To perform accurate spatial measurements, we must project coordinates from the curved three-dimensional Earth onto a flat two-dimensional map sheet. In GIS, this is managed using **Coordinate Reference Systems (CRS)**. This section details the differences between geographic and projected coordinates, datums, EPSG coding, and how to avoid coordinate transformation issues.

---

## 1. Geographic Coordinate Reference Systems (GCS)
A GCS represents the Earth as a three-dimensional sphere or ellipsoid. Location is measured in angular units (degrees) relative to the equator and the prime meridian:

* **Latitude:** Angle north or south of the Equator (varies from $-90^{\circ}$ to $+90^{\circ}$).

* **Longitude:** Angle east or west of the Prime Meridian (varies from $-180^{\circ}$ to $+180^{\circ}$).

* **Datums:** Ellipsoids that approximate the shape of the Earth. The global standard datum is **WGS 84** (used by GPS systems).

* **The Distance Dilemma:** Because longitude lines converge at the poles, the physical distance of $1^{\circ}$ of longitude changes depending on latitude. Near the equator, $1^{\circ}$ is roughly $111\text{ km}$, but near the poles, it approaches $0\text{ km}$. 

> [!WARNING]
> Because GCS uses degrees as units, running calculations like buffer distances or slopes directly on a GCS layer will yield incorrect results.

---

## 2. Projected Coordinate Reference Systems (PCS)
A PCS projects the curved 3D Earth surface onto a flat 2D plane. It uses linear units (meters or feet) instead of degrees, making it suitable for calculating distances, areas, and slopes.

```text
      3D Ellipsoid (Earth)                 2D Flat Projection
         (Lat / Lon)                           (Meters)
            __--__                              +------+
         /         \     ---Projection--->      |      |
         \  WGS84  /                            | UTM  |
           `--__--'                             +------+
```

### Map Projection Types

* **Conformable:** Preserves local angles and shapes (ideal for navigation).

* **Equal Area:** Preserves areas (ideal for mapping watershed boundaries and soil zones).

* **Equidistant:** Preserves distances along specific lines.

---

## 3. Universal Transverse Mercator (UTM)
The most common PCS for regional applications is the UTM system. UTM divides the Earth into 60 vertical zones, each $6^{\circ}$ of longitude wide.

* **Linear Units:** Locations are defined as Eastings and Northings measured in meters.

* **Zone Selection:** Each zone has its own central meridian to minimize distortion within that section.

* **UTM Zones for Nepal:**

  * **UTM Zone 44N:** Covers Western and Central Nepal (longitude $78^{\circ}\text{E}$ to $84^{\circ}\text{E}$).

  * **UTM Zone 45N:** Covers Eastern Nepal (longitude $84^{\circ}\text{E}$ to $90^{\circ}\text{E}$).

---

## 4. EPSG Codes
To simplify database management, the **European Petroleum Survey Group (EPSG)** compiled a registry of geographic and projected coordinate reference systems, assigning a unique identifier code to each.

| EPSG Code | Registry Name | CRS Type | Primary Application |
| :--- | :--- | :--- | :--- |
| **4326** | WGS 84 | Geographic (GCS) | Global data sharing, web mapping (Leaflet/Mapbox), GPS tracklogs. |
| **32644** | WGS 84 / UTM Zone 44N | Projected (PCS) | Hydrological analysis and mapping in Western/Central Nepal. |
| **32645** | WGS 84 / UTM Zone 45N | Projected (PCS) | Hydrological analysis and mapping in Eastern Nepal. |

---

## 5. Reprojection and CRS Transformations
In QGIS, datasets with different CRS can be loaded into the same map canvas. QGIS handles this automatically using **On-the-Fly (OTF) Reprojection**, transforming coordinates dynamically to match the project CRS.

* **Project CRS vs. Layer CRS:** The project CRS determines the coordinate grid of the map canvas. The layer CRS is the coordinate system stored in the source file.

* **Permanent Reprojection:** While OTF reprojection is useful for visualization, running processing tools (like clipping or buffering) on layers with mismatching coordinate systems can cause errors. You should reproject layers to a common projected coordinate system using the **Reproject Layer** tool before performing spatial analysis.

---

## 6. Common CRS Pitfalls in Projects

* **Shifted Layers:** If coordinate datasets do not align correctly (e.g., a river layer appears shifted 200 meters to the side of a DEM), it is usually because the layer datum was defined incorrectly in the metadata.

* **Incorrect Buffer Units:** Attempting to create a $100\text{ m}$ buffer around a river layer, but the output buffer is so large it covers the entire country. This happens when the input layer is in GCS (EPSG:4326), causing QGIS to interpret the "100" input value as $100^{\circ}$ instead of $100\text{ meters}$.
