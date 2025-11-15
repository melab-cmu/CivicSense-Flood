# **CivicSense** **Flood Focus Portion of the Urban Risk Intelligence Twin** **ArcGIS, Cesium for Omniverse, and NVIDIA PhysX Fluid Simulation**

### *CMU – AI Engineering for Digital Twins – Final Group Project*

## **1\. Overview**

CivicSense–FloodTwin is a **geospatially accurate, physics-driven Digital Twin** prototype that simulates urban flooding in Buellton, CA using:

* **ArcGIS Pro flood models** → water depth & velocity rasters  
* **Cesium for Omniverse** → Earth-scale geospatial context  
* **NVIDIA Omniverse** → USD-native environment  
* **PhysX GPU Fluid Particles** → interactive water–building collisions

The system demonstrates how a city can **sense, visualize, and reason** about flood risk — supporting Water Managers, planners, and emergency operations (evacuation routing, infrastructure impact, hazard awareness).

## **2\. System Architecture**

*NOTE: Insert `docs/architecture/system_architecture.png`)*

## **3\. Features**

* **Raster-driven hydrodynamics**  
  * Flood depth controls particle spawn height and density  
  * Flood extent controls emitter masks  
  * Velocity raster optionally sets initial particle push vector  
* **PhysX fluid simulation**  
  * GPU-accelerated particles  
  * Material interaction against buildings and terrain  
  * Adjustable solver substeps, particle radius, friction  
* **Cesium geospatial precision**  
  * Terrain aligned with global coordinate frame  
  * Buildings dropped into accurate world coordinates  
* **Point Cloud → USD support**  
  * E57/LAS → USD conversion  
  * Used for urban details or to improve placement accuracy

## **4\. Repository Layout**

Insert folder tree from above, or reference `/docs/repo_structure.md`.

## **5\. Installation & Requirements**

### **Software Versions**

* ArcGIS Pro 3.x  
* NVIDIA Omniverse USD Composer 2023+  
* Cesium for Omniverse Extension  
* Omniverse PhysX 5.x  
* Python 3.10+ (only for conversion scripts)

## **6\. Data Setup**

### **⚠️ Large Files Not Included**

This repo includes **sample rasters only**. Full export instructions are in `/arcgis/README.md`.

### **Download Instructions**

1. Run ArcGIS simulation  
2. Export:  
   * `DEM.tif`  
   * `Depth.tif`  
   * `Velocity.tif`  
   * `Extent.tif`  
3. Place under: /data/rasters\_local/2025-XX-XX/

## **7\. Running the Omniverse Scene**

### **Steps**

1. Open `omniverse/usd/base_scene.usda` in USD Composer  
2. Enable Cesium extension  
3. Load `cesium/ion/asset_manifest.json`  
4. Enable all buildings and point clouds  
5. Enable PhysX scene  
6. Play timeline; particles spawn based on raster sampling  
7. Use camera bookmarks to view:  
   * City overview  
   * PeaSoup  
   * Preschool

## **8\. OmniGraph: Raster → Particle Workflow**

Insert diagram from `/docs/architecture/raster_to_particles_flow.png`

### **Summary**

* Rasters converted to textures  
* OmniGraph samples UV and depth  
* Spawns particles where Depth \> threshold  
* Height \= DEM elevation \+ Depth  
* Velocity optional

## **9\. Sprint Plan**

Full sprint plan in: /docs/sprint/sprint\_plan.md  
Daily log templates in: /docs/sprint/daily\_logs/

## **10\. Limitations**

* Not a CFD model; PhysX fluids approximate shallow water  
* Performance constrained by particle count  
* Rasters must be preprocessed for identical resolution

## **11\. Next Steps**

* Add Isaac Sim vehicles & pedestrian agents  
* Implement evacuation routing layer  
* Link to real-time data (rain gauges, sensors)  
* Replace particle fluids with heightfield hydrodynamics (future)

## **12\. License**

MIT License.
