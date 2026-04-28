# Environment Setup Guide

This guide walks you through setting up the QuikDel environment from scratch.

---

## 1. Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| Python | ≥ 3.9 | 3.10 or 3.11 recommended |
| GDAL / ogr2ogr | ≥ 3.4 | Required for `.shp → .geojson` conversion |
| GraphHopper | Self-hosted or API key | For distance/time matrix generation |
| Google Account | — | Required for Google Colab + Drive storage |

---

## 2. Installing GDAL

GDAL must be installed at the **system level** before installing `geopandas`.

### Ubuntu / Debian
```bash
sudo apt-get update
sudo apt-get install -y gdal-bin libgdal-dev python3-gdal
```

### macOS (Homebrew)
```bash
brew install gdal
```

### Windows
Download the GDAL binaries from [GISInternals](https://www.gisinternals.com/release.php) and add to your PATH.

---

## 3. Python Environment

```bash
# Create and activate a virtual environment
python -m venv quikdel-env
source quikdel-env/bin/activate       # Linux/macOS
quikdel-env\Scripts\activate          # Windows

# Install dependencies
pip install -r requirements.txt
```

---

## 4. Google Drive Folder Structure

The notebooks use Google Drive for data storage. Create the following folder structure in your Drive before running:

```
📁 MyDrive/
└─ 📁 DeliverAI Data Folder/
   ├─ 📁 Columbus - RL Delivery Data/
   │   ├─ 📁 Census Data/
   │   ├─ 📁 Hotspot Data/
   │   │   └─ 📁 Data-<min_children>-<ratio>/
   │   ├─ 📁 Q Tables/
   │   │   └─ 📁 Data-<min_children>-<ratio>/
   │   └─ 📁 Results/
   │       └─ 📁 Data-<min_children>-<ratio>/
   ├─ 📁 Philadelphia - RL Delivery Data/   (same structure)
   ├─ 📁 Chicago - RL Delivery Data/        (same structure)
   └─ 📁 Raw GeoJsons/
```

Update the `personal_dir` variable at the top of each notebook to point to your `DeliverAI Data Folder/`.

---

## 5. GraphHopper Setup

QuikDel uses GraphHopper to generate distance and travel-time matrices between hotspots.

**Option A — Self-hosted (recommended for large cities):**
```bash
# Download GraphHopper
wget https://github.com/graphhopper/graphhopper/releases/download/9.1/graphhopper-web-9.1.jar

# Download an OSM extract for your city/state from https://download.geofabrik.de/
# Example for Ohio:
wget https://download.geofabrik.de/north-america/us/ohio-latest.osm.pbf

# Start the server
java -jar graphhopper-web-9.1.jar server config-example.yml
# Server runs at http://localhost:8989 by default
```

**Option B — GraphHopper Cloud API:**  
Obtain an API key from [graphhopper.com](https://www.graphhopper.com/) and set the endpoint in the Data Collection notebook.

---

## 6. U.S. Census TIGER/Line Shapefiles

Download the 2018 census tract shapefiles from:  
[https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html)

Place the downloaded `.shp` files in `📁 Raw GeoJsons/` on your Drive. The QCense notebook will handle conversion to GeoJSON using `ogr2ogr`.

---

## 7. Verifying the Installation

```python
import geopandas as gpd
import overpass
import alphashape
print("All core packages loaded successfully.")
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `ImportError: geopandas` | Ensure GDAL is installed system-wide before `pip install geopandas` |
| GraphHopper timeout | Increase the timeout parameter in the Data Collection notebook; use a local GH server for large cities |
| Overpass API rate limit | Add `time.sleep()` calls between queries or use a local Overpass mirror |
| Drive mount fails in Colab | Re-run the `drive.mount()` cell and re-authorize |
