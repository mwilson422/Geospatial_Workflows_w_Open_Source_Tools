# Attribute Joins

## Overview
Combining tabular data with spatial data based on common attributes (non-spatial joins), or joining attributes between spatial datasets. This is essential for enriching geospatial data with external information like census data, property values, demographic statistics, or any other tabular information.

## When to Use This
- Add census data to geographic boundaries (e.g., population to suburbs)
- Join property values to parcel geometries
- Combine survey data with location features
- Enrich spatial data with database records
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
