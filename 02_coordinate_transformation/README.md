# Coordinate Transformation

## Overview
Reprojecting geospatial data from one coordinate reference system (CRS) to another.

## When to Use This
- Combining datasets that use different coordinate systems
- Preparing data for web mapping (typically requires EPSG:4326 or EPSG:3857)
- Converting between geographic (lat/lon) and projected coordinate systems
- Ensuring accurate distance/area measurements (projected CRS)
- Standardizing data across a project
- Meeting client or regulatory requirements for specific coordinate systems

## Common Coordinate Systems

| EPSG Code | Name | Type | Use Case |
|-----------|------|------|----------|
| 4326 | WGS 84 | Geographic | GPS, web maps (Leaflet), global data |
| 3857 | Web Mercator | Projected | Web maps (Google, OpenStreetMap) |
| 3395 | WGS 84 / World Mercator | Projected | Worldwide mapping |
| 3577 | GDA 94 / Australian Albers | Projected | Area-based analysis across multiple states, equal area (meters) |
| 7856 | GDA 2020 / MGA Zone 56 | Projected | Sydney, eastern NSW, SE QLD |
| 3112 | GDA 94 / Geoscience Australia Lambert | Projected | General Australia wide mapping | 

**Finding EPSG codes:**
- Search at [epsg.io](https://epsg.io/)
- Search at [spatialreference.org](https://spatialreference.org/)

## Method 1: Using ogr2ogr (Command Line)
### Basic Reprojection
```bash
# Reproject to ESPG 4326 (WSG84)
ogr2ogr -t_srs EPSG:4326 output.shp input.shp
```
- `-t_srs` = target spatial reference system

### Specify source CRS (if not defined in the file)
```bash
# If source CRS is missing or incorrect
ogr2ogr -s_srs EPSG:27700 -t_srs EPSG:4326 output.shp input.shp
```

### Assign CRS Without Reprojecting
```bash
# Just assign/define the CRS (no transformation)
ogr2ogr -a_srs EPSG:4326 output.shp input.shp
```
**Purpose of Assigning CRS without transforming**
1. Most common reason is to fix missing CRS metadata
    - This might happen if the .prj file is not included with a shapefile or you get csv data with X/Y columns but no CRS info.
2. Correcting incorrectly defined CRS
    - The file has the wrong CRS metadata but the coordinates themselves are correct.
    - This means sometimes you need to investigate the data and compare the CRS given with the X/Y values given and if they are what you expect them to be. 

The coordinates do not change, you are just assigning the CRS.

## Method 2: Using Python with GeoPandas
```python
import geopandas as gpd

# Read data
gdf = gpd.read_file("input.shp")

# Check current CRS
print(f"Current CRS: {gdf.crs}")

# Reproject to WGS84
gdf_wgs84 = gdf.to_crs("EPSG:4326")

# Reproject to Web Mercator
gdf_webmerc = gdf.to_crs("EPSG:3857")

# Save reprojected data
gdf_wgs84.to_file("output.geojson", driver="GeoJSON")
```

### Handle Missing CRS
```python
# Check if CRS is defined
if gdf.crs is None:
    print("Warning: No CRS defined")
    # Assign CRS without transforming
    gdf = gdf.set_crs("EPSG:4326")
else:
    # Reproject
    gdf = gdf.to_crs("EPSG:3857")
```

### Batch Reprojection
```python
from pathlib import Path

input_dir = Path("input_data")
output_dir = Path("output_data")
output_dir.mkdir(exist_ok=True)

target_crs = "EPSG:4326"

for file in input_dir.glob("*.shp"):
    gdf = gpd.read_file(file)
    
    # Skip if already in target CRS
    if gdf.crs == target_crs:
        print(f"Skipping {file.name} - already in {target_crs}")
        continue
    
    # Reproject
    gdf_reprojected = gdf.to_crs(target_crs)
    
    # Save
    output_file = output_dir / f"{file.stem}_wgs84.geojson"
    gdf_reprojected.to_file(output_file, driver="GeoJSON")
    print(f"Reprojected: {file.name} -> {output_file.name}")
```

## Understanding CRS Types

### Geographic CRS (lat/lon)
- Units: degrees
- Used for: Global datasets, GPS, web mapping
- Examples: EPSG:4326 (WGS84)
- ⚠️ **Don't use for** distance/area calculations

### Projected CRS
- Units: meters or feet
- Used for: Accurate measurements, local/regional analysis
- Examples: UTM zones, State Plane, National Grids
- ✅ **Use for** distance/area calculations

### When to Use Which?

**Use Geographic (EPSG:4326) when:**
- Displaying on web maps (Leaflet, Mapbox GL)
- Storing GPS coordinates
- Global datasets
- Interoperability is key

**Use Projected CRS when:**
- Measuring distances or areas
- Buffer operations
- Local/regional analysis
- Accurate cartographic display

**Use Web Mercator (EPSG:3857) when:**
- Web applications (use 4326 for data storage, 3857 for display)