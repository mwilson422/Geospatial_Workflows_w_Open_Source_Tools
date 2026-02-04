# Spatial Filtering & Clipping

## Overview
Extracting features from a dataset based on their spatial relationship to a boundary or area of interest. This is essential for subsetting large datasets to specific study areas, extracting data within administrative boundaries, or focusing analysis on particular regions.

## When to Use This
- Extract features within a specific geographic area (e.g., "give me all roads in Sydney")
- Clip data to a study area boundary
- Remove features outside a region of interest
- Create regional subsets from national datasets
- Extract data within a bounding box
- Filter features that intersect, contain, or are contained by another geometry
- Prepare data for analysis by removing irrelevant areas

## Spatial Relationship Types

| Predicate | Description | Example Use |
|-----------|-------------|-------------|
| **intersects** | Any overlap or touching | "Roads that cross this suburb" |
| **within** | Completely inside | "Properties fully within the flood zone" |
| **contains** | Completely contains another | "Suburbs that contain this park" |
| **overlaps** | Partial overlap (not within/contains) | "Parcels overlapping zone boundary" |
| **touches** | Share boundary, no overlap | "Adjacent parcels" |
| **crosses** | Lines crossing polygons | "Pipelines crossing the reserve" |
| **disjoint** | No spatial relationship | "Properties not in the heritage area" |

## Method 1: Using ogr2ogr (Command Line)

### Clip to Polygon Boundary
```bash
# Basic clipping
ogr2ogr -clipsrc boundary.shp output.shp input.shp

# Clip and convert format
ogr2ogr -f "GeoJSON" -clipsrc boundary.geojson output.geojson input.shp

# Clip specific layer from GeoPackage
ogr2ogr -clipsrc study_area.gpkg output.gpkg input.gpkg layer_name
```
