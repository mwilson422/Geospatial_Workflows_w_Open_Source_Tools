# Geometry Validation & Repair

## Overview
Detecting and fixing invalid geometries in spatial data. Invalid geometries can cause errors in spatial operations, incorrect analysis results, and software crashes. This workflow helps identify, diagnose, and repair geometric problems.

## When to Use This
- Data fails to load or process correctly
- Spatial operations (buffer, intersection, union) produce errors
- Geometries appear corrupted in GIS software
- Preparing data for strict validation requirements (e.g., topology rules)
- Quality control before publishing datasets
- Fixing legacy or imported data with geometry issues
- Troubleshooting "TopologyException" or similar errors

## What Makes a Geometry Invalid?

### Common Validity Issues

| Issue | Description | Visual | Impact |
|-------|-------------|--------|--------|
| **Self-intersection** | Polygon boundary crosses itself | Figure-8 shape | Most operations fail |
| **Unclosed rings** | Polygon doesn't close | Open boundary | Cannot calculate area |
| **Duplicate points** | Consecutive identical points | Multiple points at same location | Precision issues |
| **Spike/Gore** | Very narrow triangular protrusion | Needle-like extension | Topology errors |
| **Ring order** | Exterior/interior rings wrong order | Hole as outer ring | Area calculations wrong |
| **Collapsed geometry** | Line with 2 identical points, polygon with <3 unique points | Degenerate shape | Operations fail |
| **Multi-geometries with invalid parts** | One component is invalid | Mixed valid/invalid | Partial failures |
| **Inverted hole** | Hole wound in wrong direction | Clockwise instead of counter-clockwise | Rendering issues |
