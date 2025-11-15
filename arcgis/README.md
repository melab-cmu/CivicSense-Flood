# Buellton 100-Year Flood Scenario

### ArcGIS Pro Workflow for CivicSense Hydrologic Modeling

**Version:** v01
**Project:** CivicSense – Flood Intelligence Module
**Goal:** Generate aligned, WGS84-ready rasters for downstream 3D visualization, AI agents, RAG pipelines, and Unreal/Cesium integration.

---

## 1. Overview

This workflow produces four authoritative hydrologic rasters for the 100-year storm scenario in Buellton, CA:

* **DEM.tif** (bare-earth, 32-bit float)
* **Depth.tif** (flood depth, 32-bit float)
* **Velocity.tif** (flow velocity magnitude, 32-bit float)
* **Extent.tif** (binary flood mask, 8-bit unsigned, values 0/1)

All rasters are:

* Aligned by **common snap grid and origin**
* Resampled to a unified **cell size**
* Reprojected to **WGS84 (EPSG:4326)**
* Validated through a fresh map QC pass

These outputs are standardized for CivicSense ingestion across:
CesiumJS, Unreal Engine, NVIDIA Omniverse, NIM RAG workflows, and downstream AI agents.

---

## 2. Directory Structure

```
CivicSense/
  └── arcgis/
        └── BuelltonFlood_100yr/
             ├── data/
             │     ├── raw/
             │     ├── intermediate/
             │     └── final/
             │           ├── DEM.tif
             │           ├── Depth.tif
             │           ├── Velocity.tif
             │           └── Extent.tif
             ├── gis/
             │     └── BuelltonFlood_100yr.gdb
             └── docs/
                   └── flood_modeling/
                        └── README_arcgis_buellton_100yr.md
```

---

## 3. Required Inputs

* **Source DEM** (LiDAR-derived or USGS bare earth)
* **Rainfall forcing** for 100-yr / 7-day scenario (IFD tables, county data, or prior CRF inputs)
* **2D flood model** (Arc Hydro Flood tools or imported HEC-RAS/other model outputs)
* **ArcGIS Pro with Spatial Analyst** enabled

---

## 4. Step-by-Step Workflow

### 4.1 Create new project

1. ArcGIS Pro → New → Map
2. Project name: `CivicSense_BuelltonFlood_100yr`
3. Create folders:

   * `data/raw`
   * `data/intermediate`
   * `data/final`
   * `gis/`
4. Create workspace geodatabase:
   `gis/BuelltonFlood_100yr.gdb`

---

## 4.2 DEM Preparation (bare earth → hydraulically conditioned)

1. Add DEM to map (`data/raw`).

2. Set geoprocessing environments:

   * Output CRS: **projected CRS** (e.g., StatePlane CA V or UTM zone)
   * Cell Size: same as DEM
   * Snap Raster: DEM
   * Extent: Same as DEM

3. **Fill sinks**

   * Tool: *Hydrology → Fill*
   * Output: `DEM_filled` (in GDB)

4. **Flow Direction**

   * Output: `FlowDir`

5. Export hydraulically conditioned DEM

   * Tool: *Raster to Other Format*
   * Output: `data/final/DEM_modelCRS.tif`
   * Pixel Type: `32_BIT_FLOAT`

---

## 4.3 Flood Model Run or Import

### If running internally (Arc Hydro 2D / ArcGIS tools):

* Use `DEM_filled` + rainfall forcing.
* Output rasters into GDB as:

  * `Depth_raw`
  * `Velocity_raw`

### If importing external model (e.g., HEC-RAS 2D):

1. Export model rasters as GeoTIFF (in model CRS).
2. ArcGIS → *Copy Raster* into GDB:

   * Depth → `Depth_raw`
   * Velocity → `Velocity_raw`
3. If only water surface elevation is provided:

   * Raster Calculator:

     ```
     Depth_raw = "WSE" - "DEM_filled"
     ```

---

## 4.4 Generate Flood Extent Mask

1. Threshold depth to define inundation

   * Raster Calculator:

     ```
     Con("Depth_raw" > 0.05, 1, 0)
     ```
   * Output: `Extent_raw`

2. Convert to 8-bit integer

   * *Copy Raster* → Pixel Type: `8_BIT_UNSIGNED`
   * Output: `Extent_int`

---

## 4.5 Enforce Alignment and Snap (Model CRS)

Set environments:

* Snap Raster = `DEM_filled`
* Cell Size = Same as Snap
* Extent = DEM_filled

Create aligned copies:

* `DEM_modelAligned` (32-bit float)
* `Depth_modelAligned` (32-bit float)
* `Velocity_modelAligned` (32-bit float)
* `Extent_modelAligned` (8-bit unsigned)

---

## 4.6 Reproject to WGS84 (EPSG:4326)

Use **Project Raster** for each aligned raster:

| Raster                | Method           | Pixel Type     | Resampling |
| --------------------- | ---------------- | -------------- | ---------- |
| DEM_modelAligned      | → DEM_WGS84      | 32-bit float   | Bilinear   |
| Depth_modelAligned    | → Depth_WGS84    | 32-bit float   | Bilinear   |
| Velocity_modelAligned | → Velocity_WGS84 | 32-bit float   | Bilinear   |
| Extent_modelAligned   | → Extent_WGS84   | 8-bit unsigned | Nearest    |

Ensure consistent angular cell size (e.g., **0.00001°**).

---

## 4.7 Export Final GeoTIFFs

Export from GDB → `data/final/`:

```
DEM.tif         (32-bit float)
Depth.tif       (32-bit float)
Velocity.tif    (32-bit float)
Extent.tif      (8-bit unsigned, 0/1)
```

Confirm:

* Correct CRS (WGS84)
* Correct pixel type
* Correct extent & cell size across all four outputs

---

## 5. Quality Control (QC)

1. Create a **new empty map**.

2. Add:

   * DEM.tif
   * Depth.tif
   * Velocity.tif
   * Extent.tif

3. Validate:

   * All layers snap perfectly.
   * No shifts or mismatched extents.
   * `Extent == 1` only where `Depth > threshold`.
   * `Velocity > 0` only inside flooded areas.
   * DEM values correspond to known Buellton topography.

---

## 6. Versioning Convention

Each simulation run increments the version:

```
data/final/v01/
data/final/v02/
...
```

Each version documents:

* Model CRS used
* Source DEM
* Flood return period assumptions
* Rainfall forcing sources
* Projection details
* Any preprocessing variations


## 7. Downstream Integration

After QC, these rasters flow into:

| Platform                     | Usage                                                |
| ---------------------------- | ---------------------------------------------------- |
| **CesiumJS**                 | Terrain overlays, flood visualization, time sliders  |
| **Unreal Engine 5.5**        | Flood depth shaders, velocity-driven particle fields |
| **NVIDIA Omniverse**         | USD-based hydrology layers, sensor simulation        |
| **NIM + RAG + Pinecone**     | AI flood reasoning for CivicSense agents             |
| **City Planning Dashboards** | Zoning impact, evacuation routing, risk scoring      |

This forms the hydrologic backbone of the CivicSense Flood Intelligence System.

---

## 8. Citation / Metadata Template

Include metadata per raster:

```
Event: 100-year, 7-day rainfall scenario
Location: Buellton, California
DEM Source: <dataset name + date>
Simulation Tool: <Arc Hydro / HEC-RAS / custom>
Projection Pipeline: modelCRS → WGS84 (EPSG:4326)
Processing Date: <YYYY-MM-DD>
Operator: Mel Torres, CMU AI Engineering
```

---

## Contact

For CivicSense development, research, or collaboration inquiries:
**Mel Torres — AI Engineering for Digital Twins**
GitHub: `melab-cmu`
Project: CivicSense – Smart Water & Flood Intelligence



