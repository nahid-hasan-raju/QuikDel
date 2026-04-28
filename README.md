# QuikDel: A Scalable Hierarchical Multi-Agent System for Urban Delivery Using Q-Learning

[![Python](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

This repository contains the full simulation codebase for **QuikDel**, a distributed, hierarchical multi-agent delivery routing system that uses Q-learning to optimize last-mile urban delivery. QuikDel is evaluated across three U.S. cities — Columbus, Philadelphia, and Chicago — and achieves up to a **25% reduction in total delivery distance** compared to a point-to-point baseline while handling over **13,866 delivery requests per hour**.

> **Submitted paper:** "QuikDel: A Scalable Hierarchical Multi-agent System for Urban Delivery Using Q-Learning"  
> *IEEE Access, 2024*  

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Pipeline Overview](#pipeline-overview)
- [Quick Start](#quick-start)
- [Notebooks](#notebooks)
- [Configuration](#configuration)
- [Data Format](#data-format)
- [Results Summary](#results-summary)
- [Citation](#citation)

---

## Overview

QuikDel addresses the scalability limitations of existing RL-based delivery systems (e.g., DeliverAI) by introducing:

- **QCense** — a data aggregation tool that automatically extracts census tract boundaries, producer locations (restaurants, cafes, etc.), and consumer locations (residences, offices, etc.) for any U.S. city using the U.S. Census Bureau and OpenStreetMap.
- **Hierarchical Overlay Network** — census tracts are grouped into clusters, each managed by a algorithmically selected *superspot*. This reduces Q-table size from O(N³) to O(N/k)³ + k·O(k³), enabling city-scale deployment.
- **Path Sharing** — multiple delivery orders traveling compatible routes share vehicles, reducing total distance traveled. A city-specific sharing threshold (Γ_ps) is empirically determined to avoid detour overhead on short-haul deliveries.

---

## Repository Structure

```
QuikDel/
│
├── notebooks/
│   ├── 01_QCense_Data_Extraction.ipynb       # Extract census + OSM producer/consumer data
│   ├── 02_Data_Collection_Master.ipynb        # Build hotspot network + GraphHopper matrices
│   ├── 03_Q_Table_Generation.ipynb            # Train Q-tables for all agents
│   ├── 04_Simulation.ipynb                    # Run QuikDel + baseline simulations
│   └── 05_Results_Analysis.ipynb             # Aggregate and visualize results
│
├── data/
│   └── sample/                                # Tiny sample data for quick testing
│
├── docs/
│   ├── SETUP.md                               # Full environment setup guide
│   └── DATA_FORMAT.md                         # Intermediate file format reference
│
├── assets/                                    # Figures and diagrams from the paper
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Pipeline Overview

The simulation pipeline runs sequentially across five notebooks:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Step 1: QCense Data Extraction  (Notebook 01)                              │
│  ─ Input:  U.S. Census TIGER shapefiles + OpenStreetMap (Overpass API)      │
│  ─ Output: census_tract_data.geojson  (tract polygons + producer/consumer)  │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────┐
│  Step 2: Data Collection Master  (Notebook 02)                              │
│  ─ Input:  census_tract_data.geojson                                        │
│  ─ Builds: hotspot overlay network, superspot clustering                    │
│  ─ Calls:  GraphHopper API for distance/time matrices                       │
│  ─ Output: graphhopper_dataframe.csv, distance/time .npy matrices           │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────┐
│  Step 3: Q-Table Generation  (Notebook 03)                                  │
│  ─ Input:  Hotspot data + distance/time matrices                            │
│  ─ Trains: one intra-cluster Q-table per superspot, one inter-cluster table │
│  ─ Output: Q_boltzman.npz, index mapping JSONs, tract group JSONs           │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────┐
│  Step 4: Simulation  (Notebook 04)                                          │
│  ─ Input:  All hotspot + Q-table data                                       │
│  ─ Simulates: QuikDel (with path sharing), Baseline 1, Baseline 2          │
│  ─ Output: result JSON files per city/ratio/load                            │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────┐
│  Step 5: Results Analysis  (Notebook 05)                                    │
│  ─ Input:  Result JSON files                                                │
│  ─ Output: Summary tables, distance/time plots, Γ_ps threshold plots       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/<your-org>/QuikDel.git
cd QuikDel
```

### 2. Set up the environment

```bash
python -m venv quikdel-env
source quikdel-env/bin/activate      # Windows: quikdel-env\Scripts\activate
pip install -r requirements.txt
```

> For GDAL and GraphHopper setup, see [`docs/SETUP.md`](docs/SETUP.md).

### 3. Prepare Google Drive

Create the folder structure described in [`docs/SETUP.md`](docs/SETUP.md) and download Census TIGER/Line shapefiles for your target state.

### 4. Open notebooks in Google Colab

Upload the notebooks from the `notebooks/` folder to Colab, or open them directly:

| Notebook | Open in Colab |
|---|---|
| 01 QCense Data Extraction | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) |
| 02 Data Collection Master | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) |
| 03 Q-Table Generation | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) |
| 04 Simulation | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) |
| 05 Results Analysis | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) |

### 5. Configure your city and ratio

At the top of each notebook, set:

```python
state_name = "Ohio"          # U.S. state name (full name)
city_name  = "Columbus"      # City name
superspot_hotspot_ratio = 10 # Cluster size ratio (5, 10, or 15)
```

Run notebooks in order: **01 → 02 → 03 → 04 → 05**.

---

## Notebooks

### `01_QCense_Data_Extraction.ipynb` — QCense Tool
Implements the **QCense** data aggregation tool. Given a U.S. state and city name, it:
- Fetches census tract boundaries from the U.S. Census Bureau TIGER/Line shapefiles
- Converts shapefiles to GeoJSON using `ogr2ogr`
- Queries OpenStreetMap (via Overpass API) for all producer locations (restaurants, cafes, fast food, bakeries, supermarkets, food courts) and consumer locations (residential buildings, apartments, hotels, offices, universities, hospitals)
- Overlays producers/consumers onto census tract polygons using GeoPandas
- Removes empty tracts and fills boundary holes using a convex hull strategy
- Outputs `census_tract_data.geojson`

**Key parameters:**
```python
state_name = "Illinois"
city_name  = "Chicago"
superspot_hotspot_ratio = 5   # Used for image output directory naming only here
```

---

### `02_Data_Collection_Master.ipynb` — Hotspot Network Builder
Builds the two-level hierarchical overlay network and collects routing data:
- **Superspot selection:** Scores each census tract by producer count, consumer count, and number of bordering tracts (SSE score), then selects top-scoring, non-adjacent tracts as superspots
- **Cluster assignment:** BFS from each superspot; small clusters are merged into neighbours
- **GraphHopper queries:** Calls GraphHopper routing API for road distances and travel times between all (superspot↔superspot) and (hotspot↔hotspot within cluster) pairs
- Outputs distance/time `.npy` matrices and `graphhopper_dataframe.csv`

**Key parameters:**
```python
superspot_hotspot_ratio = 10   # Target cluster size
min_children = superspot_hotspot_ratio - 3   # Minimum cluster size
```

---

### `03_Q_Table_Generation.ipynb` — Q-Learning Agent Training
Trains all RL agents using the **epsilon-greedy Q-learning** algorithm:
- One **intra-cluster Q-table** per superspot (agents navigate deliveries within a cluster)
- One **inter-cluster Q-table** (agents navigate between superspots)
- Reward function penalizes staying in place (−10), rewards reaching destination (+1), and penalizes detours (−0.1)
- Uses Boltzmann (softmax) exploration policy
- Outputs compressed Q-tables (`Q_boltzman.npz`) and all index-mapping JSON files

**Hyperparameters (Table 3 from paper):**
```python
alpha   = 0.1    # Learning rate
gamma   = 0.99   # Discount factor
epsilon = 0.99   # Exploration rate (decays during training)
```

---

### `04_Simulation.ipynb` — QuikDel Delivery Simulation
Runs the full QuikDel delivery simulation alongside both baselines:
- Generates delivery requests using a **Bimodal Gaussian** load distribution (peaks at lunch and dinner hours)
- Assigns deliveries a time limit: `min{x ∈ {15,20,25,30,45,60,90,120,180,240} | x ≥ 1.5 × direct_time}`
- **QuikDel (R):** 4-step routing with RL agents and path sharing for long-haul deliveries (d ≥ Γ_ps × dist_max)
- **Baseline 1:** Point-to-point routing, no path sharing, no hierarchical network
- **Baseline 2:** Hierarchical routing without path sharing (measures overhead of hierarchical structure)
- Outputs per-run result JSON files to `Results/`

---

### `05_Results_Analysis.ipynb` — Results & Visualization
Aggregates all simulation results and generates the paper's figures:
- Success rate, total distance, mean delivery time, vehicle count, average hops
- Γ_ps threshold plots (total distance vs. minimum sharing percentage)
- Cross-city and cross-ratio comparison tables

---

## Configuration

| Parameter | Default | Description |
|---|---|---|
| `state_name` | `"Ohio"` | Full U.S. state name |
| `city_name` | `"Columbus"` | City name |
| `superspot_hotspot_ratio` | `10` | Average number of hotspots per cluster (5, 10, or 15) |
| `min_children` | `ratio - 3` | Minimum cluster size before merging |
| `mini` | `False` | Use a small census tract subset for quick testing |
| `load` | `300` | Peak delivery load (orders/hour) |
| `Γ_ps` | Empirical | Minimum delivery distance fraction to enable path sharing |

Cities tested in the paper:

| City | State | Census Tracts | Hotspots |
|---|---|---|---|
| Columbus | Ohio | 278 | 278 |
| Philadelphia | Pennsylvania | 423 | 423 |
| Chicago | Illinois | 863 | 863 |

---

## Data Format

See [`docs/DATA_FORMAT.md`](docs/DATA_FORMAT.md) for a complete description of every intermediate file, NumPy array shape, and JSON schema.

---

## Results Summary

| City | Ratio | Success Rate | Distance Reduction vs. B1 | Avg. Delivery Time Increase |
|---|---|---|---|---|
| Columbus | 1:10 | 97.3% | 3.8% | +133 s |
| Philadelphia | 1:5 | 93.2% | 11.8% | +278 s |
| Chicago | 1:10 | 86.5% | 21.7% | +216 s |

Path sharing reduces total travel distance by up to **25%** (Chicago, 1:15 ratio) at the cost of a modest **6.5-minute** average delivery time increase.

---

## Citation

If you use QuikDel or QCense in your research, please cite:



---

## License

This project is released under the [MIT License](LICENSE).
