# Sample Data

This directory contains a minimal sample dataset for **Columbus, Ohio** (30-tract subset) that allows you to test the full pipeline without collecting data from scratch.

## Contents

```
data/sample/
├── census_tract_data_columbus_mini.geojson   # 30-tract GeoJSON subset
├── graphhopper_dataframe_mini.csv            # Pre-built hotspot dataframe
├── distance_adjacency_matrix_mini.npy        # 30×30 distance matrix
├── time_adjacency_matrix_mini.npy            # 30×30 time matrix
└── map_geoid_index_mini.json                 # GEOID → index mapping
```

## Usage

In each notebook, set:

```python
city_name = "Columbus"
mini = True
```

This will point all file paths to the mini dataset, allowing a complete end-to-end test run in minutes rather than hours.

> **Note:** Sample data covers only 30 of Columbus's 278 census tracts and is intended for **testing only**. Simulation results from the mini dataset do not reflect paper results.
