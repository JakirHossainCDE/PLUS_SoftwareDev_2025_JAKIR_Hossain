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

# Part 2: Visualization with Folium
Folium is a powerful Python library that enables the creation of interactive Leaflet maps. This capability is crucial for "FindMyRoute" as it allows us to visually present the optimized routes, selected attractions, and relevant Points of Interest to the user in an engaging and easily understandable format.

```bash
import folium
from folium.plugins import MarkerCluster # Useful for grouping many markers

print("\n--- Starting Folium Visualization ---")

# --- 1. Create a base Folium map centered on the location ---
# We calculate the centroid of the street network graph by averaging the x and y coordinates
# of all its nodes. This provides a good central point for the map.
# OSMnx graph nodes have 'x' (longitude) and 'y' (latitude) attributes.
node_xs = [data['x'] for node, data in G.nodes(data=True)]
node_ys = [data['y'] for node, data in G.nodes(data=True)]
latitude = sum(node_ys) / len(node_ys)
longitude = sum(node_xs) / len(node_xs)

m = folium.Map(location=[latitude, longitude], zoom_start=14)
print(f"Created Folium map centered at: {latitude:.4f}, {longitude:.4f}")

# --- 2. Add the street network to the map ---
# Convert the graph to a GeoDataFrame of edges and then to GeoJSON.
# This is a robust way to plot the network on Folium, bypassing specific OSMnx plotting functions
# that might change between library versions.
# Relevant documentation for graph to GeoDataFrame: https://osmnx.readthedocs.io/en/stable/user_guide.html#convert-graphs-to-geodataframes
print("Adding street network to map by converting to GeoJSON...")
edges_gdf = ox.graph_to_gdfs(G, nodes=False) # Get edges as GeoDataFrame
folium.GeoJson(
    edges_gdf.__geo_interface__, # Convert GeoDataFrame to GeoJSON dictionary
    name="Street Network",
    style_function=lambda x: {
        "color": "#8b0000",
        "weight": 2,
        "opacity": 0.7,
    },
).add_to(m)

# --- 3. Add POIs to the map ---
# To manage potentially many POIs, we'll use `MarkerCluster` from Folium plugins.
# This groups nearby markers, making the map less cluttered at lower zoom levels.
# We'll create separate clusters for different POI types for better organization and toggleability.
print("Adding POIs to map...")
marker_cluster_attractions = MarkerCluster(name="Attractions").add_to(m)
marker_cluster_cafes = MarkerCluster(name="Cafes").add_to(m)
marker_cluster_parks = MarkerCluster(name="Parks").add_to(m)
# For any other POIs that don't fit specific categories
marker_cluster_other = MarkerCluster(name="Other POIs").add_to(m)

# Iterate through each row in the POIs GeoDataFrame
for idx, row in pois.iterrows():
    # Extract coordinates. Geometry can be Point, Polygon, etc. For Polygons, we use the centroid.
    if row['geometry'] and row['geometry'].geom_type in ['Point', 'Polygon', 'MultiPolygon']:
        try:
            lon, lat = (row['geometry'].centroid.x, row['geometry'].centroid.y) if row['geometry'].geom_type != 'Point' else (row['geometry'].x, row['geometry'].y)

            # Construct a popup message for the marker, including the name and type
            popup_text = f"<b>{row.get('name', 'N/A')}</b><br>"
            if row.get('tourism'): popup_text += f"Type: {row['tourism']}<br>"
            if row.get('amenity'): popup_text += f"Type: {row['amenity']}<br>"
            if row.get('leisure'): popup_text += f"Type: {row['leisure']}<br>"
            if row.get('shop'): popup_text += f"Type: {row['shop']}<br>" # Added for more general POIs

            # Assign different icon colors and names based on the POI type
            # Font Awesome icons (prefix='fa') are commonly used with Folium.
            icon_color = "darkblue"
            icon_name = "map-marker" # Default icon
            add_to_cluster = marker_cluster_other # Default cluster

            if 'tourism' in row and row['tourism'] == 'attraction':
                icon_color = "red"
                icon_name = "star"
                add_to_cluster = marker_cluster_attractions
            elif 'amenity' in row and row['amenity'] == 'cafe':
                icon_color = "green"
                icon_name = "coffee"
                add_to_cluster = marker_cluster_cafes
            elif 'leisure' in row and row['leisure'] == 'park':
                icon_color = "lightgreen"
                icon_name = "tree"
                add_to_cluster = marker_cluster_parks

            # Create a Folium Marker and add it to the appropriate cluster
            folium.Marker(
                location=[lat, lon],
                popup=folium.Popup(popup_text, max_width=300), # Max width helps with long text
                icon=folium.Icon(color=icon_color, icon=icon_name, prefix='fa')
            ).add_to(add_to_cluster)
        except Exception as e:
            # Catch potential errors if a geometry is malformed or unexpected
            print(f"Skipping POI '{row.get('name', 'Unnamed')}' due to geometry error: {e}")
            continue

# Add a layer control to the map, allowing users to toggle different POI clusters and base layers
folium.LayerControl().add_to(m)

print("Map created with street network and categorized POIs. Displaying map...")

# --- 4. Display the map ---
# In a Jupyter environment, simply calling the map object will display it inline.
m

```
