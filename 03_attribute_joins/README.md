# Attribute Joins

## Overview
Combining tabular data with spatial data based on common attributes (non-spatial joins), or joining attributes between spatial datasets. This is essential for combining geospatial data with external information like census data, property values, demographic statistics, or any other tabular information by joining on a common field. 

## Examples of When to Use This
- Add census data to geographic boundaries (e.g., population to suburbs)
- Join property values to parcel geometries
- Combine survey data with location features
- Combine spatial data with database records
- Link addresses to point geometries
- Merge multiple attribute tables into spatial data

### Attribute Joins (Non-Spatial)
Joining based on a **common field** (like database joins)

### Spatial Joins
Joining based on **spatial relationship** (covered in spatial filtering)

This README focuses on **attribute joins**

## Join Types

| Join Type | Description | Result |
|-----------|-------------|--------|
| **Inner** | Only matching records from both | Rows where key exists in both |
| **Left** | All from left, matching from right | All left rows, NaN for non-matches |
| **Right** | All from right, matching from left | All right rows, NaN for non-matches |
| **Outer** | All records from both | All rows from both, NaN for non-matches |

## Method 1: Using ogr2ogr (Command Line)
Can do attribute joins using SQL but there are limitations and it works best when the datasets are within the same file (such as in the same GeoPakage)

```bash
# Both layers in same GeoPackage
ogr2ogr -dialect SQLite \
  -sql "SELECT parcels.*, values.property_value, values.assessment_year 
        FROM parcels 
        LEFT JOIN property_values AS values 
        ON parcels.parcel_id = values.parcel_id" \
  output.gpkg input.gpkg
```
### Use ogr2ogr when:
- Doing a simple join where both layers are in the same GeoPackage


## Method 2: Using Python

### Basic Attribute Join
```python
import geopandas as gpd
import pandas as pd

# Read spatial data
suburbs = gpd.read_file("suburbs.gpkg")
print(suburbs.columns)
# ['SUBURB_ID', 'SUBURB_NAME', 'geometry']

# Read tabular data (CSV, Excel, database, etc.)
population = pd.read_csv("population.csv")
print(population.columns)
# ['SUBURB_ID', 'POPULATION', 'MEDIAN_AGE', 'MEDIAN_INCOME']

# Join on common field
suburbs_enriched = suburbs.merge(
    population,
    on='SUBURB_ID',
    how='left'  # Keep all suburbs, even if no population data
)

print(suburbs_enriched.columns)
# ['SUBURB_ID', 'SUBURB_NAME', 'geometry', 'POPULATION', 'MEDIAN_AGE', 'MEDIAN_INCOME']

# Save result
suburbs_enriched.to_file("suburbs_with_population.gpkg")
```

### Join with Different Column Names
```python
# Spatial data has 'SUBURB_CODE', CSV has 'CODE'
suburbs_enriched = suburbs.merge(
    population,
    left_on='SUBURB_CODE',
    right_on='CODE',
    how='left'
)

# Remove duplicate column
suburbs_enriched = suburbs_enriched.drop(columns=['CODE'])
```

### Multiple Column Join
```python
# Join on multiple fields (composite key)
suburbs_enriched = suburbs.merge(
    population,
    left_on=['STATE', 'SUBURB_NAME'],
    right_on=['STATE_CODE', 'SUBURB'],
    how='inner'
)
```

### Join Types Comparison
```python
# Inner join - only matching records
inner = suburbs.merge(population, on='SUBURB_ID', how='inner')
print(f"Inner join: {len(inner)} records")

# Left join - all suburbs, NaN for missing population
left = suburbs.merge(population, on='SUBURB_ID', how='left')
print(f"Left join: {len(left)} records")
print(f"Suburbs without data: {left['POPULATION'].isna().sum()}")

# Right join - all population records, might lose some suburbs
right = suburbs.merge(population, on='SUBURB_ID', how='right')
print(f"Right join: {len(right)} records")

# Outer join - everything from both
outer = suburbs.merge(population, on='SUBURB_ID', how='outer')
print(f"Outer join: {len(outer)} records")
```

## Common Issues & Troubleshooting
- Data types are different (int vs string)
- Values don't match (whitespace, case sensitivity)

**Solutions:**
```python
# Check unique values in both datasets
print("Spatial keys (first 10):", suburbs['SUBURB_ID'].head(10).tolist())
print("Data keys (first 10):", population['SUBURB_ID'].head(10).tolist())

# Check data types
print("Spatial type:", suburbs['SUBURB_ID'].dtype)
print("Data type:", population['SUBURB_ID'].dtype)

# Convert types if needed
suburbs['SUBURB_ID'] = suburbs['SUBURB_ID'].astype(str)
population['SUBURB_ID'] = population['SUBURB_ID'].astype(str)

# Clean text fields
suburbs['SUBURB_ID'] = suburbs['SUBURB_ID'].str.strip()
population['SUBURB_ID'] = population['SUBURB_ID'].str.strip()
```

### Whitespace and Case Issues
```python
# Clean up text fields before joining
population['SUBURB_NAME'] = (
    population['SUBURB_NAME']
    .str.strip()           # Remove leading/trailing whitespace
    .str.upper()           # Convert to uppercase
    .str.replace('  ', ' ') # Remove double spaces
)

suburbs['SUBURB_NAME'] = (
    suburbs['SUBURB_NAME']
    .str.strip()
    .str.upper()
    .str.replace('  ', ' ')
)

# Now join
suburbs_enriched = suburbs.merge(population, on='SUBURB_NAME')
```

## Performance Tips

1. **Clean data before joining**
```python
   # Remove duplicates and nulls first
   population = population.dropna(subset=['SUBURB_ID'])
   population = population.drop_duplicates(subset='SUBURB_ID')
```

2. **Select only needed columns**
```python
   # Don't load entire CSV if you only need a few columns
   population = pd.read_csv(
       "census.csv",
       usecols=['SUBURB_ID', 'POPULATION', 'MEDIAN_AGE']
   )
```

3. **Use appropriate join type**
```python
   # Inner join is faster than outer join
   # Use inner if you only want matching records
   result = suburbs.merge(population, on='SUBURB_ID', how='inner')
```

4. **Convert to same data type before joining**
```python
   # Type conversion during join is slow
   # Convert beforehand
   suburbs['SUBURB_ID'] = suburbs['SUBURB_ID'].astype(str)
   population['SUBURB_ID'] = population['SUBURB_ID'].astype(str)
```

5. **Index join keys for large datasets**
```python
   # Set index on join key for faster joining
   population = population.set_index('SUBURB_ID')
   result = suburbs.join(population, on='SUBURB_ID')
```

