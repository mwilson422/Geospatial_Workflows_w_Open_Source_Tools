# Format Conversion

## Overview
Converting geospatial data between different formats (Shapefile, GeoJSON, GeoPackage, KML, etc.)

## When to Use This
- Sharing data with systems that require specific formats
- Preparing data for web applications (GeoJSON)
- Standardizing datasets from multiple sources
- Converting legacy formats to modern standards

## Method 1: Using ogr2ogr (Command Line)

### Shapefile to GeoJSON
```bash
ogr2ogr -f "GeoJSON" ouptput.geojson input.shp
```
The -f flag uses the driver name. The common ones are:

**Vector Formats**
```bash 
-f "ESRI Shapefile"    # .shp
-f "GeoJSON"           # .geojson, .json
-f "GPKG"              # .gpkg (GeoPackage)
-f "KML"               # .kml
-f "CSV"               # .csv
-f "GML"               # .gml
-f "SQLite"            # .sqlite, .db
-f "PostgreSQL"        # PostGIS database
-f "FileGDB"           # .gdb (Esri File Geodatabase)
-f "FlatGeobuf"        # .fgb
-f "GeoJSONSeq"        # .geojsonl (newline-delimited)
-f "MapInfo File"      # .tab
-f "DXF"               # .dxf
-f "TopoJSON"          # .topojson
```

To find all available formats:
```bash
ogrinfo --formats
```

To get detailed info about a specific driver:
```bash
ogrinfo --format "GeoJSON"
```

To see all ogr2ogr options
```bash
ogr2ogr --help
```

## Additional Options
**Common ogr2ogr Flags**
- `-t_srs EPSG:4326` = target spatial reference system (will reproject to this)
- `-a_srs EPSG:2193` = assigns a spatial reference system (without reprojecting)
- `-nlt POLYGON` = force the geometry type (POINT, LINESTRING, POLYGON, etc.)
- `-dim XY` = Force 2D (strip Z/M values)
- `-append` = append to existing layer
- `-progress` = show progress bar

All flags can be found in ogr2ogr docs:
[ogr2ogr reference](https://gdal.org/en/stable/programs/ogr2ogr.html)


## Method 2: Using Python and GeoPandas

```python
import geopandas as gpd

# Read shapefile
gdf = gpd.read_file("input.shp")

# Convert to GeoJSON
gdf.to_file("output.geojson", driver="GeoJSON")

# Convert to GeoPackage
gdf.to_file("output.gpkg", layer="layer_name", driver="GPKG")
```

## Notes
GeoPandas uses Fiona (which wraps GDAL/OGR), so it supports the same formats as using GDAL/OGR. 