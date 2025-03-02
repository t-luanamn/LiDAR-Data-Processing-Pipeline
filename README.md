# LiDAR Data Processing Pipeline: From Raw Data to Digital Elevation Model (DEM)

This document outlines a pipeline designed to process raw **LiDAR** (Light Detection and Ranging) data, ultimately generating a **Digital Elevation Model** (DEM). The pipeline cleans the data, isolates ground points, and interpolates them to create a raster representation of the terrain.

# 🎯 Pipeline Overview

The pipeline performs the following key operations:

## 1. **Data Input:**

The pipeline begins by reading the `.laz` file, which contains raw point cloud data.

## 2. **Ground Point Extraction:**

To generate an accurate DEM, the pipeline filters out non-ground points, retaining only those classified as ground `(Class 2)`.

```json
{
	"type": "filters.range",
	"limits": "Classification[2:2]"
}
```

- Extracts only ground points (Classification = 2).
- LiDAR point clouds classify surfaces into ground (2), buildings, water, vegetation, etc.
- Keeping only ground points is useful for terrain analysis and DEM generation.

## 3. **Outlier Removal**

Removes noise and isolated points that are likely to be errors (e.g., birds, vehicles, errors in scanning).

```json
{
    "type": "filters.outlier",
    "method": "radius",
    "radius": 1.0,
    "min_k": 3
}
```

- radius: 1.0 → Only keeps points that have neighbors within 1.0 unit.
- min_k: 3 → If a point has fewer than 3 neighbors, it's removed as noise.

## 4. **Ground Point Output:**

The cleaned ground points are written to a new `.las` file for further analysis and visualization.

```json
{
    "type": "writers.las",
    "filename": "output_ground.las"
}
```

- Writes the cleaned LiDAR ground points to a new file (output_ground.las).
- Storing the filtered data allows further analysis or visualisation.
- .las format is widely used in GIS and geospatial applications.

## 5. **DEM Generation:**

Creates a DEM (in `.tif` format) from the filtered ground points.

```json
{
    "type": "writers.gdal",
    "filename": "output_dem.tif",
    "resolution": 1.0,
    "output_type": "idw"
}
```

- resolution: 1.0 → The DEM has a 1m grid cell size.
- output_type: "idw" → Uses Inverse Distance Weighting (IDW) interpolation to estimate terrain elevation between points.
- DEMs are essential for terrain analysis, flood modeling, construction planning, and GIS applications.
- Ground points represent bare earth, making them ideal for DEM generation.

## ⚙️ Pipeline Configuration (pipeline.json)

The pipeline's steps and parameters are defined in a `pipeline.json` configuration file. Below is the structure of this file and it's content that will be referenced in the step by step pipeline description :

```json
{
	"pipeline": [
		"data/BURW_final_ULS_reproj.laz",
		{
			"type": "filters.range",
			"limits": "Classification[2:2]"
		},
		{
			"type": "filters.outlier",
			"method": "radius",
			"radius": 1.0,
			"min_k": 3
		},
		{
			"type": "writers.las",
			"filename": "output_ground.las"
		},
		{
			"type": "writers.gdal",
			"filename": "output_dem.tif",
			"resolution": 1.0,
			"output_type": "idw"
		}
	]
}
```

📂 This file serves as the blueprint for processing the `.laz` file and generating a DEM.

# 🔍 Data Inspection

To verify the quality of raw and processed data, two Jupyter notebooks have been used:

- 📌 `LAS_inspect.ipynb` – Exploring the raw `.laz` file (visualisation & analysis).
- 📌 `LiDAR_processing.ipynb` – Running the pipeline using `pipeline.json` (processing steps & output validation).

## 📌 Tools Used

- PDAL – Open-source library for LiDAR data processing
- laspy – Reads and writes .las/.laz LiDAR files in Python
- NumPy – Efficient numerical computations and array manipulations
- Pandas – Data analysis and tabular manipulation
- Matplotlib – 2D and 3D visualization of point clouds
- Axes3D – 3D plotting capabilities from mpl_toolkits.mplot3d
- rasterio – Reads and writes raster datasets (e.g., DEMs)
- JSON – Stores and configures processing pipelines
