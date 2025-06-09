# 🛰️ Assignment A3 – Geospatial Python Libraries: OSMnx and Folium
**By Jakir Hossain**

## 📌 Overview

This notebook demonstrates the use of two crucial geospatial Python libraries: **OSMnx** and **Folium**. These libraries are central to our final project, *"FindMyRoute: Discover the Optimised Tourist Path for Your City,"* as they enable the core functionalities of **data acquisition** (road networks and Points of Interest) and **interactive map visualization**.

### Objectives:

- Explain the role of OSMnx in fetching street networks and POIs from OpenStreetMap.
- Demonstrate how to use OSMnx to acquire this data for Salzburg, Austria.
- Explain the role of Folium in creating interactive web maps.
- Show how to visualize the acquired street networks and POIs on a Folium map.


---

## 🧰 Libraries Used

### OSMnx
- Purpose: Downloading, constructing, analyzing, and visualizing street networks and other geospatial data from OpenStreetMap.
- 📚 [Documentation](https://osmnx.readthedocs.io/en/stable/)
- Common functions:
  - `osmnx.graph_from_place()`
  - `osmnx.features.features_from_place()`

### Folium
- Purpose: Creating interactive Leaflet maps in Python.
- 📚 [Documentation](https://python-visualization.github.io/folium/)
- Common functions:
  - `folium.Map()`
  - `folium.PolyLine()`
  - `folium.Marker()`

---

## ⚙️ Installation

Install the necessary libraries using `pip`:

```bash
pip install osmnx folium matplotlib
```


## 🗺️ Area of Interest
We'll focus on Salzburg, Austria, a city relevant to our final project and rich in OpenStreetMap data.


# Part 1: Data Acquisition with OSMnx
OSMnx simplifies the process of interacting with OpenStreetMap (OSM) data. It allows us to easily download street networks as networkx graph objects and other geographical features (like buildings, parks, or shops) as geopandas GeoDataFrames. This is fundamental for our "FindMyRoute" project as it provides the base road network for routing and the data for identifying Points of Interest (POIs).


```bash
import osmnx as ox
import matplotlib.pyplot as plt # Often used by OSMnx for plotting, good to include
import pandas as pd # For inspecting GeoDataFrames

print("--- Starting OSMnx Data Acquisition ---")

# Define the place of interest for data download
place_name = "Salzburg, Austria"
print(f"Acquiring data for: {place_name}")

# --- 1. Download Street Network ---
# The `osmnx.graph_from_place()` function downloads the street network for the
# specified place and creates a networkx MultiDiGraph.
# The 'network_type' parameter dictates what kind of network to get.
# 'walk' includes pedestrian paths, sidewalks, and streets usable by pedestrians.
# Relevant documentation: https://osmnx.readthedocs.io/en/stable/user_guide.html#create-a-graph
print("Downloading street network...")
G = ox.graph_from_place(place_name, network_type="walk")
print(f"Street network graph created with {len(G.nodes)} nodes and {len(G.edges)} edges.")

# --- 2. Download Points of Interest (POIs) ---
# The `osmnx.features_from_place()` function allows us to query various types of
# geographic features based on their OpenStreetMap tags.
# For 'FindMyRoute', we are interested in tourist attractions, cafes, and green spaces.
# Relevant documentation: https://osmnx.readthedocs.io/en/stable/user_guide.html#retrieve-osm-amenities-and-other-features
print("Downloading Points of Interest (POIs)...")
# Define OSM tags for the types of POIs we want
tags = {"tourism": "attraction", "amenity": "cafe", "leisure": "park"}
pois = ox.features_from_place(place_name, tags)
print(f"Found {len(pois)} POIs of various types.")

# Display a sample of the POIs GeoDataFrame to see its structure and available tags.
# This helps in understanding the data we've retrieved.
print("\nSample of POIs GeoDataFrame (first 5 rows and relevant columns):")
# Modified to exclude 'osm_id' which may not always be present as a direct column,
# causing a KeyError. 'name' and 'geometry' are fundamental, and other amenity
# columns are generally present based on the defined tags.
print(pois[['name', 'geometry', 'amenity', 'tourism', 'leisure']].head())

# We can also filter the POIs for specific types if needed, for example, just cafes.
cafes = pois[pois['amenity'] == 'cafe']
print(f"\nFound {len(cafes)} cafes in the retrieved POIs.")

print("\n--- OSMnx Data Acquisition Complete ---")

```


