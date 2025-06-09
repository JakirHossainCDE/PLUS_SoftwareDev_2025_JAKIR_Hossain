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



🗺️ Area of Interest
We'll focus on Salzburg, Austria, a city relevant to our final project and rich in OpenStreetMap data.



Part 1: Data Acquisition with OSMnx


