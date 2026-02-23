# Spatial Filtering 

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

### SQL WHERE Clause Filtering
```bash
# Filter by attribute
ogr2ogr -where "POPULATION > 10000" output.shp input.shp

# Complex SQL
ogr2ogr -sql "SELECT * FROM parcels WHERE area_m2 > 5000 AND zone = 'R1'" \
  output.gpkg input.gpkg
```

**Because, ogr2ogr supports SQL queries with SQLite dialect, can use the spatial predicates with `ST_` prefix.**
```bash
# Features that intersect a boundary
ogr2ogr -dialect SQLite \
  -sql "SELECT a.* FROM input_layer a, boundary_layer b 
        WHERE ST_Intersects(a.geometry, b.geometry)" \
  output.gpkg input.gpkg
  ```


  ## Method 2: Using Python with GeoPandas

  ### Basic Clipping
```python
import geopandas as gpd

# Read data
features = gpd.read_file("buildings.shp")
boundary = gpd.read_file("study_area.geojson")

# Ensure same CRS
if features.crs != boundary.crs:
    boundary = boundary.to_crs(features.crs)

# Clip features to boundary
clipped = gpd.clip(features, boundary)

# Save result
clipped.to_file("clipped_buildings.gpkg", driver="GPKG")

print(f"Original features: {len(features)}")
print(f"Clipped features: {len(clipped)}")
```

### Spatial Join / Filtering by Predicate
```python
import geopandas as gpd

# Read datasets
points = gpd.read_file("crime_incidents.gpkg")
suburbs = gpd.read_file("suburbs.gpkg")

# Ensure same CRS
points = points.to_crs(suburbs.crs)
```
```python
# Filter: points within suburbs
# Method 1: Using sjoin (spatial join)
points_in_suburbs = gpd.sjoin(
    points, 
    suburbs, 
    how='inner',      # Join Type: 'inner', 'left', 'right'
    predicate='within'  # Spatial Relationship: 'intersects', 'within', 'contains', etc.
)
```
**Available predicates:**
- 'intersects' - any overlap or touch (default)
- 'within' - left completely inside right
- 'contains' - left completely contains right
- 'overlaps' - partial overlap
- 'crosses' - geometries cross
- 'touches' - share boundary but don't overlap

```python
# Method 2: Using overlay
# Get points that intersect suburbs (adds suburb attributes)
points_with_suburb = gpd.overlay(
    points,
    suburbs,
    how='intersection'
)

print(f"Points in suburbs: {len(points_in_suburbs)}")
```
**Difference between .sjoin and .overlay**
- **.sjoin**:
  - keeps original geometries unchanged
  - adds attributes from one dataset to antoher based on spatial relationship
  - Output has **same geometries** as input (left or right depending on the `how`)

- **.overlay**
  - creates new geometries from geometric operations
  - combines attributes from both layers
  - Output has **new geometries** created from intersection/union/etc (Only the intersecting parts)

**When to Use:**
- **.sjoin**
  - When you don't want to modify geometries (just want to know which features realte to which)
  - Tagging points with polygon attributes
    ```python
    # "Which suburb is each crime incident in?"
    crime_points = gpd.read_file("crimes.gpkg")
    suburbs = gpd.read_file("suburbs.gpkg")

    crimes_tagged = gpd.sjoin(crime_points, suburbs, predicate='within')
    # Result: Crime points with suburb_name added
    ```
  - Finding features that meet spatial criteria
    ```python
    # "Which parcels are in the flood zone?"
    parcels = gpd.read_file("parcels.gpkg")
    flood_zone = gpd.read_file("flood_zone.gpkg")

    at_risk = gpd.sjoin(parcels, flood_zone, predicate='within')
    # Result: List of parcels (with original geometries) that are in flood zone
    ```
  - Counting relationships
    ```python
    # "How many buildings per suburb?"
    buildings = gpd.read_file("buildings.gpkg")
    suburbs = gpd.read_file("suburbs.gpkg")

    joined = gpd.sjoin(buildings, suburbs, predicate='within')
    counts = joined.groupby('suburb_name').size()
    # Result: Count of buildings per suburb
    ```

- **.overlay**
  - When you want to create new geometries 
  - Splitting features by boundaries
    ```python
    # "Split parcels by zoning districts"
    parcels = gpd.read_file("parcels.gpkg")
    zones = gpd.read_file("zoning.gpkg")

    split_parcels = gpd.overlay(parcels, zones, how='intersection')
    # Result: Parcels split at zone boundaries, each piece has zone info
    ```
  - Calculating areas in different categories
    ```python
    # "How much of each parcel is in each zone?"
    result = gpd.overlay(parcels, zones, how='intersection')
    result['area_in_zone'] = result.area
    summary = result.groupby(['parcel_id', 'zone_type'])['area_in_zone'].sum()
    ```
  - Creating new geographic units
    ```python
    # "Merge overlapping features"
    merged = gpd.overlay(layer1, layer2, how='union')
    ```
  - Cookie-cutter operations
    ```python
    # "Extract only the parts of parcels that are in conservation areas"
    conservation_parcels = gpd.overlay(parcels, conservation, how='intersection')
    ```
  - Finding gaps or differences
    ```python
    # "Which areas are zoned but don't have parcels?"
    gaps = gpd.overlay(zones, parcels, how='difference')
    ```


**Set Operations for `gdp.overlay()`**
```python
parcels = gpd.read_file("parcels.gpkg")
zones = gpd.read_file("zoning.gpkg")

# Intersection: Only overlapping parts
intersection = gpd.overlay(parcels, zones, how='intersection')
# Result: Parcels split by zone boundaries, with attributes from both

# Union: All features from both
union = gpd.overlay(parcels, zones, how='union')
# Result: All parcels and zones, split at intersections

# Difference: Parcels minus zones
difference = gpd.overlay(parcels, zones, how='difference')
# Result: Parts of parcels that DON'T overlap zones

# Symmetric difference: Non-overlapping parts
sym_diff = gpd.overlay(parcels, zones, how='symmetric_difference')
# Result: Parts that are in one OR the other, but not both

# Identity: All parcels, split where zones overlap
identity = gpd.overlay(parcels, zones, how='identity')
# Result: Like intersection but keeps all of left layer
```


## Common Issues & Troubleshooting

### Problem: Different CRS causing incorrect filtering
**Symptom:** No features returned, or wrong features selected

**Solution:**
```python
# Always check and align CRS before spatial operations
print(f"Features CRS: {features.crs}")
print(f"Boundary CRS: {boundary.crs}")

if features.crs != boundary.crs:
    boundary = boundary.to_crs(features.crs)
    print("✓ CRS aligned")
```