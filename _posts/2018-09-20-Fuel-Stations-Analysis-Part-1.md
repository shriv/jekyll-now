Journeys begin with a question
> How can we measure value derived from spatial placement of amenities? 

This question landed on my lap through various chains at my work, and my fascination with figuring out how to answer it led me to the exciting (and burgeoning!) field of urban data science. It's become the thing that makes me get out of bed in the morning! 

Read on..


# Defining a business question

The question of value from amenities is an important one for the private and public sector alike. In this short series, I'll be comparing two brands of fuel station networks that are well represented in Wellington, New Zealand: Z and BP.

Since data science is about delivering value to some entity, a natural business question we can track throughout the analyses is:
> *Does Z have better coverage than their competitor(s) in Wellington? If so, how?*


# Setting the metric scene
We can approximate value as *usefulness*. For fuel station networks, usefulness can be broken down into some high level aspects:

- Fuel station locations 
-- Coverage

- Interactions of fuel stations with the environment
-- With Humans: accessibility, convenience
-- With Other entities (e.g. amenities like cafes, cinema theatres etc.): locale

- Fuel station characteristics
-- Available amenities (e.g. toilets, fuel types, food etc.) - supply
-- Uptake of availabile amenities (i.e. higher scores for amenities that are used more) - where supply meets demand

To make concrete comparisons between competing fuel station brands, we need to break down the high level aspects into proxy metrics, or quantitative analyses.

- Coverage
-- Structure of fuel station inter-connectivity
-- Average spatial separation between two fuel stations
-- Nearest neighbours: same brand or competitor? 
- Interactions with the 'human' environment
-- Driving accessibility 

# The Fuel Stations series at a glance
The above analyses are split into 3 posts since the entirity is almost unreadable:

- Part 1 (this post)
-- Introduction to spatial data
-- Getting fuel station data from OpenStreetMap

- [Part 2](https://shriv.github.io/Fuel-Stations-Analysis-Part-2/)
-- Abstracting spatial networks into networks
-- Calculating inter-connectivity metrics 

- [Part 3](https://shriv.github.io/Fuel-Stations-Analysis-Part-3/)
-- Driving accessibility analysis


# Resources
I've used studies from urban data science as inspiration. Some resources I've consulted include:
- [Talk on Urban Data Science by Dr. Cecilia Mascolo](https://www.youtube.com/watch?v=eNpdvzORWVc&t=2162s)
- [Understanding Traffic with Open Data by researchers at Oxford Internet Institute](https://www.youtube.com/watch?v=0GM0sEvQ2-M)
- [Geoff Boening's blog - especially his package OSMnx](https://geoffboeing.com/2016/11/osmnx-python-street-networks/)
- [Proximity and accessibility analyses with Pandana](https://github.com/gboeing/urban-data-science/blob/master/20-Accessibility-Walkability/pandana-accessibility-demo-full.ipynb)

The code and formalised report of the analysis can be found in a [github repo](https://github.com/shriv/fuel-stations). The text is almost identical to the blog posts.


# Introduction to Spatial data
Spatial data includes geographical information for physical entities in our world. Since we're focusing on man-made entities like fuel stations, the simplest geographical information we require is geolocation. This information can be enriched with attributes that describe local geography - derived from the surrounding environment.    

## What spatial data do we need?
We need the following data to calculate the coverage metrics and do the quantitative analyses. There are two key types of data: Base and Points of Interest (POIS). The tools used to get and process the data are described in more detail in the following section.

Data | Type | Why? | How?
--- | --- | --- | ---
Geo-tagged fuel stations | POIS | The key spatial entities we're interested in | Open Street Map via Overpass
Regional map and street network | Base | To connect the fuel stations via real world streets and roads. | Open Street Map via OSMnx
Street network broken up into regular points | Base |  For simpler interactions between geography and fuel stations | Open Street Map via Panadana

Open Street Maps (OSM) is the underlying data source. [OSM](https://en.wikipedia.org/wiki/OpenStreetMap) is an open, collaborative map of the world. Map data can be queried and downloaded locally using the [Overpass API](https://wiki.openstreetmap.org/wiki/Overpass_API). [In a later section](#Resolving-data-issues), I will show how I leveraged the collaborative feature to update / add information on Wellington fuel stations. This editing feature will continue to be an important aspect of any future version of this project, e.g. the available services at the various fuel stations. 

The main advantage of using OSM, other than the altruistic aspect of enriching the available information for others, is that the same framework provides data for *all* fuel station brands. Comparative analyses become much easier since they don't require data munging from different sources. 

## Summary of spatial analyses  
As indicated in the previous section, there are two main data types: Base and POIS. At a simplistic level, the analyses outlined in this report are essentially different types of *interactions* between the base data and POIS. I've focused on pairwise interactions involving:
- *POIS - POIS* interactions via the base layer -- Distances between fuel stations
- *Base grid - POIS* interactions -- Distances between map grid points and fuel stations

While these interactions seem trivial, they can generate considerable insight into coverage.

- *POIS - POIS* interactions via the base layer
    - Fuel station network analysis to understand connectivity 
- *Base grid - POIS* interactions
    - Accessibility of fuel stations

# Tools 
This report is generated with Python 2.7 running in a [Jupyter Notebook](http://jupyter.org/). All the Python packages highlighted in this section can be easily installed using a package manager like *conda* or *pip*. The list is not meant to be exhaustive - please refer to the explicit dependencies in the Python code. 

Tool | Type | What does it do?
--- |---| ---
Jupyter Notebook| General| Allows for the creation of an annotated, executable analysis. Like this one! 
Pandas | General| Enables data to be stored, manipulated and analysed from dataframes (like R)
Matplotlib & Seaborn | General| General and ggplot2-like plotting libraries
Networkx | General | For standard network analysis when we don't require spatial information. 
OSMnx | Spatial | For analysis of streets and roads with network algorithms.
Pandana | Spatial | For fast and efficient accessibility analyses. 


# Get Spatial data
This section is somewhat techincal. The specifics are not necessary to understand the analysis. 

## Set bounding box

A bounding box of lattitude and longitude coordinates describes a rectangular geospatial region. For this report, I've chosen a bounding box that includes Wellington City and some of Lower Hutt. This selection is important since only the entities *within* the bounding box are used in the analysis. The visual tool [here](http://boundingbox.klokantech.com/) is useful for obtaining the bounding box coordinates from a user-defined rectangle on the map. 

A key technical point is that [bounding box conventions do vary](https://wiki.openstreetmap.org/wiki/Bounding_Box
):
- The general definition uses (min Longitude , min Latitude , max Longitude , max Latitude), or  (W, S, E, N) 
- Pandana and Overpass use (S, W, N, E).

![]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/bounding_box_selection.png)


## Create Query
The following section creates a query to get fuel station data from Open Street Maps. The tags list can also be amended to get other amenities. The full list is [here](https://wiki.openstreetmap.org/wiki/Key:amenity). For example, we can easily get data for cafes and restaurants by adding these to the tags list.  

The Overpass API query is not very easy to read but the main components are: 
- The bounding box: the area where we want the search performed. 
- Data Primitives: ways, nodes, tags, relations.

The data primities of OSM have an intrinsic hierarchy with nodes being the root primitive. 
- Nodes: Single point with explicit [lat, lon] coordinates. Root primitive
- Ways: Collection of nodes that defines a polygon (e.g. a building) or polyline (e.g. a road). 
- Relations: Represent the relationship of existing nodes and ways
- Tags: Metadata stored as key-value pairs. 

The main primitives used in this report are nodes and tags. The nodes give the geolocation while we use the tags to filter specifically for fuel station nodes. More information about the entities of Open Street Maps can be found [here](https://en.wikipedia.org/wiki/OpenStreetMap#Operation). 

## Getting data from Overpass 
Getting data from Open Street Map is fairly simple via the Overpass API. All you need to do is construct the search query and reshape the result JSON into your data structure of choice. I've reshaped the data as a Pandas dataframe - basically a table with columns that contain metadata about each fuel station. 

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>lat</th>
      <th>lon</th>
      <th>name</th>
      <th>operator</th>
      <th>brand</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2845230323</td>
      <td>-41.325288</td>
      <td>174.810883</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2845230324</td>
      <td>-41.325284</td>
      <td>174.811057</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2845230322</td>
      <td>-41.325275</td>
      <td>174.810774</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2845230321</td>
      <td>-41.325200</td>
      <td>174.810729</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>node</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5821475056</td>
      <td>-41.325128</td>
      <td>174.810920</td>
      <td>Z Broadway</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>


## Get Fuel Stations
The data downloaded from OSM (via Overpass) includes _all_ nodes and ways tagged as 'fuel'. The brand of the fuel station can be be used to filter for station-specific analysis. In NZ, there are 4 retailer brands: Z, Caltex, BP and Mobil. Since the merger in 2016, Z and Caltex can be regarded as two brands from a single entity. Additional details of brand coverage [here](https://www.greaterauckland.org.nz/2016/06/09/petrol-station-shakeup/). In this preliminary version of the analysis, I've only include the explicit Z branded fuel stations. For a more general analysis of the Z *entity*, we'd also need to include the Caltex branded fuel stations. 

### Data Issues
We can query the Wellington fuel stations dataset to only get those that are associated with Z.The query returns 13 Z stations within the search region. From a cursory glance at the named Z stations and [the list from the website](https://bit.ly/2KEqu5m), we can see that there is considerable parity. We're missing *Z Constable St*, but I believe the rest are there. Despite the close parity however, there are some issues with the data:
- Inconsistency between the operator and brand attributes. 
- No geolocation for some stations. 

![]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/ways_without_geoloc.png)

The key problem with the data is that a significant portion of the stations don't have location coordinates. This problem, unfortunately stems from the two main types of OSM topological entities: ways and nodes. Depending on how a user marks out the location of a fuel station, the entity can be either a way or a node. 

- If the station is marked with a single point, the entity is a node with a clear geolocation. 
- If the station's perimeter / main building is traced out as a polygon, the entity is a way with no clear geolocation.  

### Resolving data issues
Since the underlying problem is a data issue, we can add / edit the data ourselves. You can sign up and verify your email as an OSM editor - quite easy to do. Once I got the permission to edit OSM, I simply went in and added / updated the nodes for the Z fuel stations. Later, I also updated the BP and Mobil stations in the Wellington bounding box used for the analysis. 

# Fuel stations
After editing the OSM, the corrected list of Z stations is now at parity with the Z website. BP stations can also be extracted from the downloaded OSM data. 

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>lat</th>
      <th>lon</th>
      <th>name</th>
      <th>operator</th>
      <th>brand</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5821475056</td>
      <td>-41.325128</td>
      <td>174.810920</td>
      <td>Z Broadway</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3120151445</td>
      <td>-41.320054</td>
      <td>174.794407</td>
      <td>Z Kilbirnie</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5821475059</td>
      <td>-41.314924</td>
      <td>174.813972</td>
      <td>Z Miramar</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5821475061</td>
      <td>-41.313163</td>
      <td>174.781812</td>
      <td>Z Constable Street</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5821475058</td>
      <td>-41.297146</td>
      <td>174.776556</td>
      <td>Z Taranaki Street</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5544110098</td>
      <td>-41.294501</td>
      <td>174.774397</td>
      <td>Z Vivian St</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>6</th>
      <td>5821475063</td>
      <td>-41.281636</td>
      <td>174.778417</td>
      <td>Z Harbour City</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>7</th>
      <td>5821475060</td>
      <td>-41.256020</td>
      <td>174.765535</td>
      <td>Z Crofton Downs</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2206248455</td>
      <td>-41.236226</td>
      <td>174.906171</td>
      <td>Z Seaview</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>9</th>
      <td>331132009</td>
      <td>-41.226300</td>
      <td>174.806795</td>
      <td>Z Johnsonville</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>10</th>
      <td>5821475057</td>
      <td>-41.222778</td>
      <td>174.868833</td>
      <td>Z Petone</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2118620317</td>
      <td>-41.214312</td>
      <td>174.887163</td>
      <td>Z Hutt Road</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>12</th>
      <td>5821475062</td>
      <td>-41.204023</td>
      <td>174.914085</td>
      <td>Z VIC Corner</td>
      <td>NaN</td>
      <td>Z</td>
      <td>node</td>
    </tr>
    <tr>
      <th>13</th>
      <td>319121061</td>
      <td>-41.197885</td>
      <td>174.937446</td>
      <td>Z High Street</td>
      <td>Z</td>
      <td>NaN</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>


## Visualsing fuel stations
Folium is a great package for embedding Leaflet apps into your Jupyter notebook and then directly to the web! The Z fuel stations can be visualised as an interactive map with 4 lines of code. 

Z stations | BP stations
-----------|-------------
<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVM9ZmFsc2U7IExfTk9fVE9VQ0g9ZmFsc2U7IExfRElTQUJMRV8zRD1mYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgPHN0eWxlPiNtYXBfZWU4YTM2Y2MzYjFhNGZkY2JjZGJiOTJjYTY4MWJiYWIgewogICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgdG9wOiAwLjAlOwogICAgICAgIH0KICAgIDwvc3R5bGU+CjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiIiA+PC9kaXY+CjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgIAogICAgICAgIHZhciBib3VuZHMgPSBudWxsOwogICAgCgogICAgdmFyIG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYiA9IEwubWFwKAogICAgICAgICdtYXBfZWU4YTM2Y2MzYjFhNGZkY2JjZGJiOTJjYTY4MWJiYWInLCB7CiAgICAgICAgY2VudGVyOiBbLTQxLjI5LCAxNzQuOF0sCiAgICAgICAgem9vbTogMTEsCiAgICAgICAgbWF4Qm91bmRzOiBib3VuZHMsCiAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3LAogICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgIH0pOwoKICAgIAogICAgCiAgICB2YXIgdGlsZV9sYXllcl8xMzNhM2FkMmExMzE0MzY3OTZlMGY0YzIyZDI1ODJlMSA9IEwudGlsZUxheWVyKAogICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgewogICAgICAgICJhdHRyaWJ1dGlvbiI6IG51bGwsIAogICAgICAgICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgCiAgICAgICAgIm1heE5hdGl2ZVpvb20iOiAxOCwgCiAgICAgICAgIm1heFpvb20iOiAxOCwgCiAgICAgICAgIm1pblpvb20iOiAwLCAKICAgICAgICAibm9XcmFwIjogZmFsc2UsIAogICAgICAgICJzdWJkb21haW5zIjogImFiYyIKfSkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgIAogICAgICAgIHZhciBtYXJrZXJfY2YyNzYxOWFiZWQxNDg1OTg4NDljNTQwOWU1ZGEyOGMgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMjUxMjgyLCAxNzQuODEwOTIwM10sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOGYxZmIxMjIwZjExNDZiMTlhMjc2ZGMwNWE0NTc2MmEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zOWQwMDViZTMwMjQ0Y2E5YWNlODkwOTA4MGVkMzMxOSA9ICQoJzxkaXYgaWQ9Imh0bWxfMzlkMDA1YmUzMDI0NGNhOWFjZTg5MDkwODBlZDMzMTkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogQnJvYWR3YXk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzhmMWZiMTIyMGYxMTQ2YjE5YTI3NmRjMDVhNDU3NjJhLnNldENvbnRlbnQoaHRtbF8zOWQwMDViZTMwMjQ0Y2E5YWNlODkwOTA4MGVkMzMxOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2NmMjc2MTlhYmVkMTQ4NTk4ODQ5YzU0MDllNWRhMjhjLmJpbmRQb3B1cChwb3B1cF84ZjFmYjEyMjBmMTE0NmIxOWEyNzZkYzA1YTQ1NzYyYSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzlhMTRlODk5ZmZhNDQ1MWZhY2VlNzM0ZjM3MjgyODliID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMzIwMDU0MSwgMTc0Ljc5NDQwNjldLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2FlMWRmNjBiYmEzMTQxMThhZGQ3ZDRlZDUwYzIzY2NhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYTdiNzJlNmMwYmUyNDllMTk4YjY5NTY5ZTlhNzkwNTQgPSAkKCc8ZGl2IGlkPSJodG1sX2E3YjcyZTZjMGJlMjQ5ZTE5OGI2OTU2OWU5YTc5MDU0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5aIEtpbGJpcm5pZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYWUxZGY2MGJiYTMxNDExOGFkZDdkNGVkNTBjMjNjY2Euc2V0Q29udGVudChodG1sX2E3YjcyZTZjMGJlMjQ5ZTE5OGI2OTU2OWU5YTc5MDU0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfOWExNGU4OTlmZmE0NDUxZmFjZWU3MzRmMzcyODI4OWIuYmluZFBvcHVwKHBvcHVwX2FlMWRmNjBiYmEzMTQxMThhZGQ3ZDRlZDUwYzIzY2NhKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfOGE3OWJjNGI4MGQ5NDUwOWIyNjNiYzNiYmNiNGYwZDYgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMTQ5MjQ0LCAxNzQuODEzOTcxNV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMmNjYTQwYzAwOTEwNGE0MmFlMDQxNGNiOGNhMjY3M2MgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zODU5YjViYjI0NTM0MzY1OTg3Y2QxYmQ4YjFkNWYwZiA9ICQoJzxkaXYgaWQ9Imh0bWxfMzg1OWI1YmIyNDUzNDM2NTk4N2NkMWJkOGIxZDVmMGYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogTWlyYW1hcjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMmNjYTQwYzAwOTEwNGE0MmFlMDQxNGNiOGNhMjY3M2Muc2V0Q29udGVudChodG1sXzM4NTliNWJiMjQ1MzQzNjU5ODdjZDFiZDhiMWQ1ZjBmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfOGE3OWJjNGI4MGQ5NDUwOWIyNjNiYzNiYmNiNGYwZDYuYmluZFBvcHVwKHBvcHVwXzJjY2E0MGMwMDkxMDRhNDJhZTA0MTRjYjhjYTI2NzNjKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfNTU4MDkwNDg2ZjIwNGQ5NDk2ZWM4ZGIzNTEwMjAxNTIgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMTMxNjMxLCAxNzQuNzgxODExNl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfY2NiODI1ODM2ZjM2NGQ3YzlkOTIxNzM2ODAyYjU5YzcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mMjM4YzJjYzhkOTM0MWEwYmJlNWUzNWQwM2UyZmNkYyA9ICQoJzxkaXYgaWQ9Imh0bWxfZjIzOGMyY2M4ZDkzNDFhMGJiZTVlMzVkMDNlMmZjZGMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogQ29uc3RhYmxlIFN0cmVldDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfY2NiODI1ODM2ZjM2NGQ3YzlkOTIxNzM2ODAyYjU5Yzcuc2V0Q29udGVudChodG1sX2YyMzhjMmNjOGQ5MzQxYTBiYmU1ZTM1ZDAzZTJmY2RjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTU4MDkwNDg2ZjIwNGQ5NDk2ZWM4ZGIzNTEwMjAxNTIuYmluZFBvcHVwKHBvcHVwX2NjYjgyNTgzNmYzNjRkN2M5ZDkyMTczNjgwMmI1OWM3KQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfMTJhMjBkNWZlODIyNGQwZWEzYzE1YzkyZWQwZDcwZGYgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yOTcxNDYsIDE3NC43NzY1NTYxXSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZWU4YTM2Y2MzYjFhNGZkY2JjZGJiOTJjYTY4MWJiYWIpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wODEzMjZjZjI5MGM0YTgwYjY4MTk2MWI3MGJmNmMxYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VlZTI5N2EwOGFkYjQ1ZTliZjhmN2YxODEzMmQ0ZDA5ID0gJCgnPGRpdiBpZD0iaHRtbF9lZWUyOTdhMDhhZGI0NWU5YmY4ZjdmMTgxMzJkNGQwOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+WiBUYXJhbmFraSBTdHJlZXQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzA4MTMyNmNmMjkwYzRhODBiNjgxOTYxYjcwYmY2YzFjLnNldENvbnRlbnQoaHRtbF9lZWUyOTdhMDhhZGI0NWU5YmY4ZjdmMTgxMzJkNGQwOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzEyYTIwZDVmZTgyMjRkMGVhM2MxNWM5MmVkMGQ3MGRmLmJpbmRQb3B1cChwb3B1cF8wODEzMjZjZjI5MGM0YTgwYjY4MTk2MWI3MGJmNmMxYykKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2U2N2E0ZDI5MDM0ZTQ3Njk4YjJkMzlkNmFjMDM2NTZjID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjk0NTAxLCAxNzQuNzc0Mzk3NV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWJhZWNhNWZkZGYwNDg0ZTgyZjQxYzgyOGU4Y2RiYjIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81ZDFjY2ViNTlhNTQ0MzhhOTZjNmJmOTlkNjBjMmQ0YSA9ICQoJzxkaXYgaWQ9Imh0bWxfNWQxY2NlYjU5YTU0NDM4YTk2YzZiZjk5ZDYwYzJkNGEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogVml2aWFuIFN0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85YmFlY2E1ZmRkZjA0ODRlODJmNDFjODI4ZThjZGJiMi5zZXRDb250ZW50KGh0bWxfNWQxY2NlYjU5YTU0NDM4YTk2YzZiZjk5ZDYwYzJkNGEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9lNjdhNGQyOTAzNGU0NzY5OGIyZDM5ZDZhYzAzNjU2Yy5iaW5kUG9wdXAocG9wdXBfOWJhZWNhNWZkZGYwNDg0ZTgyZjQxYzgyOGU4Y2RiYjIpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl8xNDA4Y2FmYjk2ODM0OTMwYWZhMTY0NWRhOWNmMWY2ZiA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjI4MTYzNjEsIDE3NC43Nzg0MTcxXSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZWU4YTM2Y2MzYjFhNGZkY2JjZGJiOTJjYTY4MWJiYWIpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83YTc0MTE1NTk1Mjg0MDBlOGM0NWYxYjJhOGU4NWM2ZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzAyODdkM2RiMTI2NTRlODhiZWExZWQxZTIwZWE3NjkxID0gJCgnPGRpdiBpZD0iaHRtbF8wMjg3ZDNkYjEyNjU0ZTg4YmVhMWVkMWUyMGVhNzY5MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+WiBIYXJib3VyIENpdHk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzdhNzQxMTU1OTUyODQwMGU4YzQ1ZjFiMmE4ZTg1YzZkLnNldENvbnRlbnQoaHRtbF8wMjg3ZDNkYjEyNjU0ZTg4YmVhMWVkMWUyMGVhNzY5MSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzE0MDhjYWZiOTY4MzQ5MzBhZmExNjQ1ZGE5Y2YxZjZmLmJpbmRQb3B1cChwb3B1cF83YTc0MTE1NTk1Mjg0MDBlOGM0NWYxYjJhOGU4NWM2ZCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2U1ODNkOWQwMzE5MzQ3MjliZGZkNWY5NWE1OTMyMWE5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjU2MDE5OSwgMTc0Ljc2NTUzNDddLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzdkYjM4NGRhYWNkMTQ1NWQ4OGVkY2JhMWRjYjljYmY5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfY2MyZWMzOTk2ZjE1NGQzYmI3OGE5NGM3OTQzNWViM2UgPSAkKCc8ZGl2IGlkPSJodG1sX2NjMmVjMzk5NmYxNTRkM2JiNzhhOTRjNzk0MzVlYjNlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5aIENyb2Z0b24gRG93bnM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzdkYjM4NGRhYWNkMTQ1NWQ4OGVkY2JhMWRjYjljYmY5LnNldENvbnRlbnQoaHRtbF9jYzJlYzM5OTZmMTU0ZDNiYjc4YTk0Yzc5NDM1ZWIzZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2U1ODNkOWQwMzE5MzQ3MjliZGZkNWY5NWE1OTMyMWE5LmJpbmRQb3B1cChwb3B1cF83ZGIzODRkYWFjZDE0NTVkODhlZGNiYTFkY2I5Y2JmOSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzMwODk5Y2JiM2I2MDRkNTNiYzQ2NjQ0NmRiODI3Y2MwID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjM2MjI1NSwgMTc0LjkwNjE3MTFdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY5ZWI4NGUwYTM1MTQ5NjA4MjY0NjFhNGI1NGRhNWUwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNmU3N2MxMGZjMDVmNDBmYjk0MTNlNDA3ZWQ2ZGNiMzAgPSAkKCc8ZGl2IGlkPSJodG1sXzZlNzdjMTBmYzA1ZjQwZmI5NDEzZTQwN2VkNmRjYjMwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5aIFNlYXZpZXc8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY5ZWI4NGUwYTM1MTQ5NjA4MjY0NjFhNGI1NGRhNWUwLnNldENvbnRlbnQoaHRtbF82ZTc3YzEwZmMwNWY0MGZiOTQxM2U0MDdlZDZkY2IzMCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzMwODk5Y2JiM2I2MDRkNTNiYzQ2NjQ0NmRiODI3Y2MwLmJpbmRQb3B1cChwb3B1cF82OWViODRlMGEzNTE0OTYwODI2NDYxYTRiNTRkYTVlMCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzFmNDQzMWQyNDdmOTQ5OTA5YWFjMjEwN2VlMjQ3ODI4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjI2Mjk5OSwgMTc0LjgwNjc5NTRdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzliZGVjZjljZDIwZjRkODk4MTI5MjJjODljMWUzYmY4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMmFiY2ZmMWQwZWVlNGQ5YmJkM2E0YWQyYWRiZTU0NzcgPSAkKCc8ZGl2IGlkPSJodG1sXzJhYmNmZjFkMGVlZTRkOWJiZDNhNGFkMmFkYmU1NDc3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5aIEpvaG5zb252aWxsZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOWJkZWNmOWNkMjBmNGQ4OTgxMjkyMmM4OWMxZTNiZjguc2V0Q29udGVudChodG1sXzJhYmNmZjFkMGVlZTRkOWJiZDNhNGFkMmFkYmU1NDc3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMWY0NDMxZDI0N2Y5NDk5MDlhYWMyMTA3ZWUyNDc4MjguYmluZFBvcHVwKHBvcHVwXzliZGVjZjljZDIwZjRkODk4MTI5MjJjODljMWUzYmY4KQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfZjlkYzk3Njk5NGE5NGY0NDg1ZWRlM2E0MTQ0YjQ0ZDAgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMjI3NzgyLCAxNzQuODY4ODMyNl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfODVkNzZjY2MwZDRlNGQxZGE0YmEzYTUyMmU2YmQzOTQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82NzcwNzYxYTdiNzk0MTBmYjgyMTVkY2ZlN2VlZjZlNiA9ICQoJzxkaXYgaWQ9Imh0bWxfNjc3MDc2MWE3Yjc5NDEwZmI4MjE1ZGNmZTdlZWY2ZTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogUGV0b25lPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84NWQ3NmNjYzBkNGU0ZDFkYTRiYTNhNTIyZTZiZDM5NC5zZXRDb250ZW50KGh0bWxfNjc3MDc2MWE3Yjc5NDEwZmI4MjE1ZGNmZTdlZWY2ZTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9mOWRjOTc2OTk0YTk0ZjQ0ODVlZGUzYTQxNDRiNDRkMC5iaW5kUG9wdXAocG9wdXBfODVkNzZjY2MwZDRlNGQxZGE0YmEzYTUyMmU2YmQzOTQpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl9iM2EzMmFhMzczMmE0NDQwOWI2OTUwZTI3NTIxNWE0YyA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIxNDMxMiwgMTc0Ljg4NzE2MjhdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9lZThhMzZjYzNiMWE0ZmRjYmNkYmI5MmNhNjgxYmJhYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2M5Y2U4YjBlYjYwNzQ3NTE5ZWU2YzFkMDM0MzI2NGYzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2Y2ZGQwMGNjMTBhNDczOGFiYzJlNWExZjAwNmYwNTUgPSAkKCc8ZGl2IGlkPSJodG1sXzNmNmRkMDBjYzEwYTQ3MzhhYmMyZTVhMWYwMDZmMDU1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5aIEh1dHQgUm9hZDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzljZThiMGViNjA3NDc1MTllZTZjMWQwMzQzMjY0ZjMuc2V0Q29udGVudChodG1sXzNmNmRkMDBjYzEwYTQ3MzhhYmMyZTVhMWYwMDZmMDU1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfYjNhMzJhYTM3MzJhNDQ0MDliNjk1MGUyNzUyMTVhNGMuYmluZFBvcHVwKHBvcHVwX2M5Y2U4YjBlYjYwNzQ3NTE5ZWU2YzFkMDM0MzI2NGYzKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfZWVjZjBiOTEzOGRjNDZlODgxNzFkNjlhYzUyMmVmMDkgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMDQwMjM1LCAxNzQuOTE0MDg0OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2VlOGEzNmNjM2IxYTRmZGNiY2RiYjkyY2E2ODFiYmFiKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWY2NTFmM2QwMjliNDUxMjlhNmI3MTEwNWRmZjM5MDMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80ZjIxZGFjM2U4Njk0NDcwYjliZmEzZDY1NTdiZTM3YyA9ICQoJzxkaXYgaWQ9Imh0bWxfNGYyMWRhYzNlODY5NDQ3MGI5YmZhM2Q2NTU3YmUzN2MiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlogVklDIENvcm5lcjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWY2NTFmM2QwMjliNDUxMjlhNmI3MTEwNWRmZjM5MDMuc2V0Q29udGVudChodG1sXzRmMjFkYWMzZTg2OTQ0NzBiOWJmYTNkNjU1N2JlMzdjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfZWVjZjBiOTEzOGRjNDZlODgxNzFkNjlhYzUyMmVmMDkuYmluZFBvcHVwKHBvcHVwXzVmNjUxZjNkMDI5YjQ1MTI5YTZiNzExMDVkZmYzOTAzKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfMmI4MDI1YTI3OWZjNDdlMmI4M2NhY2Y2OThiMDk3OGEgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4xOTc4ODUxLCAxNzQuOTM3NDQ2XSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZWU4YTM2Y2MzYjFhNGZkY2JjZGJiOTJjYTY4MWJiYWIpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wMGM0YTRjYzJjZTQ0NzlmYTlhYzY2NWVkZmI4ODE3OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VjNjFjYjFlYTc3NTQ0MjliNzExYWY5MTlmMjY0ZGViID0gJCgnPGRpdiBpZD0iaHRtbF9lYzYxY2IxZWE3NzU0NDI5YjcxMWFmOTE5ZjI2NGRlYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+WiBIaWdoIFN0cmVldDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMDBjNGE0Y2MyY2U0NDc5ZmE5YWM2NjVlZGZiODgxNzguc2V0Q29udGVudChodG1sX2VjNjFjYjFlYTc3NTQ0MjliNzExYWY5MTlmMjY0ZGViKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMmI4MDI1YTI3OWZjNDdlMmI4M2NhY2Y2OThiMDk3OGEuYmluZFBvcHVwKHBvcHVwXzAwYzRhNGNjMmNlNDQ3OWZhOWFjNjY1ZWRmYjg4MTc4KQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div> | 
<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVM9ZmFsc2U7IExfTk9fVE9VQ0g9ZmFsc2U7IExfRElTQUJMRV8zRD1mYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgPHN0eWxlPiNtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMgewogICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgdG9wOiAwLjAlOwogICAgICAgIH0KICAgIDwvc3R5bGU+CjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjIiA+PC9kaXY+CjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgIAogICAgICAgIHZhciBib3VuZHMgPSBudWxsOwogICAgCgogICAgdmFyIG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyA9IEwubWFwKAogICAgICAgICdtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMnLCB7CiAgICAgICAgY2VudGVyOiBbLTQxLjI5LCAxNzQuOF0sCiAgICAgICAgem9vbTogMTEsCiAgICAgICAgbWF4Qm91bmRzOiBib3VuZHMsCiAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3LAogICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgIH0pOwoKICAgIAogICAgCiAgICB2YXIgdGlsZV9sYXllcl83ZTU1YmE1Y2QyMWU0YWQwOTVkYTdkOTBkYWZiZDY3NSA9IEwudGlsZUxheWVyKAogICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgewogICAgICAgICJhdHRyaWJ1dGlvbiI6IG51bGwsIAogICAgICAgICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgCiAgICAgICAgIm1heE5hdGl2ZVpvb20iOiAxOCwgCiAgICAgICAgIm1heFpvb20iOiAxOCwgCiAgICAgICAgIm1pblpvb20iOiAwLCAKICAgICAgICAibm9XcmFwIjogZmFsc2UsIAogICAgICAgICJzdWJkb21haW5zIjogImFiYyIKfSkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgIAogICAgICAgIHZhciBtYXJrZXJfMTE1NGM2MTc5ODgyNDM4YmE1MjBmMDkxMWIyNmU0YzEgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMTk2NzE5LCAxNzQuNzc1NjQ2OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGRmY2UxNTUzODY5NGQxNWJjYWI1ZmY1MzMzNzYyNTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85YjFmMTc1ZjU4ZWU0MjNhOGFlZjk1OTlhZGMwZGU0NSA9ICQoJzxkaXYgaWQ9Imh0bWxfOWIxZjE3NWY1OGVlNDIzYThhZWY5NTk5YWRjMGRlNDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEJlcmhhbXBvcmU8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBkZmNlMTU1Mzg2OTRkMTViY2FiNWZmNTMzMzc2MjU4LnNldENvbnRlbnQoaHRtbF85YjFmMTc1ZjU4ZWU0MjNhOGFlZjk1OTlhZGMwZGU0NSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzExNTRjNjE3OTg4MjQzOGJhNTIwZjA5MTFiMjZlNGMxLmJpbmRQb3B1cChwb3B1cF8wZGZjZTE1NTM4Njk0ZDE1YmNhYjVmZjUzMzM3NjI1OCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzM5ODA2NGU1NTBlNjQ0MzE5ZjQxZjJjNjNhYTAwNDExID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMzA2NzMxMSwgMTc0Ljc2MzM2OTVdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MzOTA4ZWExNjc2YzQzMmM4ZjA0NTZhNjMxZGNhYTdmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2U2ZDAyNTY2ZTEyNDU4NjhiNTZlYTAwZWRlZmRmYzAgPSAkKCc8ZGl2IGlkPSJodG1sXzdlNmQwMjU2NmUxMjQ1ODY4YjU2ZWEwMGVkZWZkZmMwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBCcm9va2x5bjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzM5MDhlYTE2NzZjNDMyYzhmMDQ1NmE2MzFkY2FhN2Yuc2V0Q29udGVudChodG1sXzdlNmQwMjU2NmUxMjQ1ODY4YjU2ZWEwMGVkZWZkZmMwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMzk4MDY0ZTU1MGU2NDQzMTlmNDFmMmM2M2FhMDA0MTEuYmluZFBvcHVwKHBvcHVwX2MzOTA4ZWExNjc2YzQzMmM4ZjA0NTZhNjMxZGNhYTdmKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfODZjYmQxYjU3MzliNGI2NWI2OTk3YzhlNjA1NjBlYTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMDMwMzYxLCAxNzQuNzc5NjA1Ml0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTQ0NTA2MGRmZmMxNGE0ZWJlODExNWIzOTEyMTIxMWUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yODZkYzNkOTJjYzY0ZjIyOTY0Y2E0M2VjNjRkOTQyYSA9ICQoJzxkaXYgaWQ9Imh0bWxfMjg2ZGMzZDkyY2M2NGYyMjk2NGNhNDNlYzY0ZDk0MmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEFkZWxhaWRlIFJvYWQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzU0NDUwNjBkZmZjMTRhNGViZTgxMTViMzkxMjEyMTFlLnNldENvbnRlbnQoaHRtbF8yODZkYzNkOTJjYzY0ZjIyOTY0Y2E0M2VjNjRkOTQyYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzg2Y2JkMWI1NzM5YjRiNjViNjk5N2M4ZTYwNTYwZWExLmJpbmRQb3B1cChwb3B1cF81NDQ1MDYwZGZmYzE0YTRlYmU4MTE1YjM5MTIxMjExZSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2Y3ZjUwNjdhNjEzNTQzMDhhNDE0M2E5ODA0M2I2ZmY3ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjkwOTUxLCAxNzQuNzgwMjEyNl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNmVjYjhhMzNhMDZiNGVhNzk0MTcyYjgzZWY4ODVjZjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MzcxZTlmMDQ4MzM0Njc4OGY3M2Y1ODUzYzM0M2NkZCA9ICQoJzxkaXYgaWQ9Imh0bWxfNzM3MWU5ZjA0ODMzNDY3ODhmNzNmNTg1M2MzNDNjZGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFJvYWRtYXN0ZXI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzZlY2I4YTMzYTA2YjRlYTc5NDE3MmI4M2VmODg1Y2YwLnNldENvbnRlbnQoaHRtbF83MzcxZTlmMDQ4MzM0Njc4OGY3M2Y1ODUzYzM0M2NkZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2Y3ZjUwNjdhNjEzNTQzMDhhNDE0M2E5ODA0M2I2ZmY3LmJpbmRQb3B1cChwb3B1cF82ZWNiOGEzM2EwNmI0ZWE3OTQxNzJiODNlZjg4NWNmMCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2MyMzhmYzNkYTM5NzQ0YjY4NWRkYjViMWJhY2IyOTYzID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjg0ODU4OCwgMTc0LjczNjM5NjVdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzlkNjZkYjE1YWFiYzRmZTE5MjY2Y2Y1YWY3MzQ0YTkyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzEyMGZkMTQ2ZTg5NDcxNjk5NjU4Y2ZkZGM4MDlhYmMgPSAkKCc8ZGl2IGlkPSJodG1sX2MxMjBmZDE0NmU4OTQ3MTY5OTY1OGNmZGRjODA5YWJjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBLYXJvcmk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzlkNjZkYjE1YWFiYzRmZTE5MjY2Y2Y1YWY3MzQ0YTkyLnNldENvbnRlbnQoaHRtbF9jMTIwZmQxNDZlODk0NzE2OTk2NThjZmRkYzgwOWFiYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2MyMzhmYzNkYTM5NzQ0YjY4NWRkYjViMWJhY2IyOTYzLmJpbmRQb3B1cChwb3B1cF85ZDY2ZGIxNWFhYmM0ZmUxOTI2NmNmNWFmNzM0NGE5MikKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzUyYjU3MTU3ZWUyNzQ2Y2Q5MjNlOTcxNWUzMjc3MjM5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjYyNjcyOCwgMTc0Ljk0NjE0MTFdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzU2YjMxZjU1YjVmMjRmM2I5ZjgwM2MwYzczZGMzNjIyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDJhNzdmMTkzODlhNDNiYjk3ZjAxM2Q1YTJjOGJkNGYgPSAkKCc8ZGl2IGlkPSJodG1sX2QyYTc3ZjE5Mzg5YTQzYmI5N2YwMTNkNWEyYzhiZDRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBXYWludWlvbWF0YTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTZiMzFmNTViNWYyNGYzYjlmODAzYzBjNzNkYzM2MjIuc2V0Q29udGVudChodG1sX2QyYTc3ZjE5Mzg5YTQzYmI5N2YwMTNkNWEyYzhiZDRmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTJiNTcxNTdlZTI3NDZjZDkyM2U5NzE1ZTMyNzcyMzkuYmluZFBvcHVwKHBvcHVwXzU2YjMxZjU1YjVmMjRmM2I5ZjgwM2MwYzczZGMzNjIyKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfYzc5NDVjZDk0ODU0NGEyNWE1Y2VkYzRhODlkMjkzMzIgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yNDEyMTU5LCAxNzQuOTA4MTc4OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjcxM2MyYjhlOWUyNDc5NTllZjE2NzFhNTNiYjI1Y2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jY2QyYzlmZTdlMDM0ZmNmYmZmYWJjZGRjMzBmMWZjNiA9ICQoJzxkaXYgaWQ9Imh0bWxfY2NkMmM5ZmU3ZTAzNGZjZmJmZmFiY2RkYzMwZjFmYzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFNlYXZpZXcgVHJ1Y2tzdG9wPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mNzEzYzJiOGU5ZTI0Nzk1OWVmMTY3MWE1M2JiMjVjYi5zZXRDb250ZW50KGh0bWxfY2NkMmM5ZmU3ZTAzNGZjZmJmZmFiY2RkYzMwZjFmYzYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9jNzk0NWNkOTQ4NTQ0YTI1YTVjZWRjNGE4OWQyOTMzMi5iaW5kUG9wdXAocG9wdXBfZjcxM2MyYjhlOWUyNDc5NTllZjE2NzFhNTNiYjI1Y2IpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl81NmUwNjczMTgzMGY0ZThhYWIzMWViNjQ2MTc5MzM0ZSA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIzMjA4ODcsIDE3NC44Mzc0MDYyXSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lNmYzNjg5OWUwNzA0Njk0YjFiZmU2YWYzOGE5NzM4OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VlNDQ4YjVlZDY2ZTRjNmI5NWQxYjFhNjJkMTcyOTFjID0gJCgnPGRpdiBpZD0iaHRtbF9lZTQ0OGI1ZWQ2NmU0YzZiOTVkMWIxYTYyZDE3MjkxYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgSHV0dCBSb2FkPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lNmYzNjg5OWUwNzA0Njk0YjFiZmU2YWYzOGE5NzM4OC5zZXRDb250ZW50KGh0bWxfZWU0NDhiNWVkNjZlNGM2Yjk1ZDFiMWE2MmQxNzI5MWMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl81NmUwNjczMTgzMGY0ZThhYWIzMWViNjQ2MTc5MzM0ZS5iaW5kUG9wdXAocG9wdXBfZTZmMzY4OTllMDcwNDY5NGIxYmZlNmFmMzhhOTczODgpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl85NDdjOTc4Y2YxY2E0OGVkOTJkYTNkZmQwOGVkYjNjZiA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIyOTc1NTksIDE3NC44MTc2MjA0XSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yMmRjNzIxMWFmZjQ0MzQxYTdmOWZiM2E2MmM5ZTAyMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUxYzQ4ODhiYmM4NjRlNzU5ODc3OGQ0MDEzMjJlMDYzID0gJCgnPGRpdiBpZD0iaHRtbF81MWM0ODg4YmJjODY0ZTc1OTg3NzhkNDAxMzIyZTA2MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgTmV3bGFuZHM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzIyZGM3MjExYWZmNDQzNDFhN2Y5ZmIzYTYyYzllMDIwLnNldENvbnRlbnQoaHRtbF81MWM0ODg4YmJjODY0ZTc1OTg3NzhkNDAxMzIyZTA2Myk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzk0N2M5NzhjZjFjYTQ4ZWQ5MmRhM2RmZDA4ZWRiM2NmLmJpbmRQb3B1cChwb3B1cF8yMmRjNzIxMWFmZjQ0MzQxYTdmOWZiM2E2MmM5ZTAyMCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzQzNzcwY2Q0NGJjYjQ5MzBhNjE3ODVjMTU1YTdhM2VmID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjI1ODU3LCAxNzQuODA3NDkyMl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMTAzOWQzNmFiYmRiNGNjYzg4ODM5OTE1NjQ4ZDJlNmQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85ZDMzOWJkZjVlNDQ0ZTc1YjBhNjVlMmVjYWJhZDM2ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfOWQzMzliZGY1ZTQ0NGU3NWIwYTY1ZTJlY2FiYWQzNmYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEpvaG5zb252aWxsZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTAzOWQzNmFiYmRiNGNjYzg4ODM5OTE1NjQ4ZDJlNmQuc2V0Q29udGVudChodG1sXzlkMzM5YmRmNWU0NDRlNzViMGE2NWUyZWNhYmFkMzZmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNDM3NzBjZDQ0YmNiNDkzMGE2MTc4NWMxNTVhN2EzZWYuYmluZFBvcHVwKHBvcHVwXzEwMzlkMzZhYmJkYjRjY2M4ODgzOTkxNTY0OGQyZTZkKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfOTQ2NGRmMTc1OTkwNDIyYjkxM2MwNDQyOGZjODJlYTMgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMjI4Mjg1LCAxNzQuOTExOTYyMl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNGE0ODg0NGFkNjY5NGQ4NWI3ZjQ1MjY2YWRmZjVmM2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lYjFhNDc5Nzk0MjY0NThjYTEyODgxNjIyMGE2YmM2OCA9ICQoJzxkaXYgaWQ9Imh0bWxfZWIxYTQ3OTc5NDI2NDU4Y2ExMjg4MTYyMjBhNmJjNjgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFdhaXdoZXR1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80YTQ4ODQ0YWQ2Njk0ZDg1YjdmNDUyNjZhZGZmNWYzYi5zZXRDb250ZW50KGh0bWxfZWIxYTQ3OTc5NDI2NDU4Y2ExMjg4MTYyMjBhNmJjNjgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl85NDY0ZGYxNzU5OTA0MjJiOTEzYzA0NDI4ZmM4MmVhMy5iaW5kUG9wdXAocG9wdXBfNGE0ODg0NGFkNjY5NGQ4NWI3ZjQ1MjY2YWRmZjVmM2IpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl81NDY2MmQyYzA2NzI0OTgzOWRjNGYxNTU4NmVlMzYzYSA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIxMjYwNzEsIDE3NC44OTM2NTQ4XSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zNDViZGUwZWJlMmY0ODE4OGIxYjczYTk0NzkyMGQwNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE4YjU4MjBmZTdlYjQ2MzZiMDgzNjUwMmQ5OTVlZDU2ID0gJCgnPGRpdiBpZD0iaHRtbF8xOGI1ODIwZmU3ZWI0NjM2YjA4MzY1MDJkOTk1ZWQ1NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgUmFpbHdheTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMzQ1YmRlMGViZTJmNDgxODhiMWI3M2E5NDc5MjBkMDUuc2V0Q29udGVudChodG1sXzE4YjU4MjBmZTdlYjQ2MzZiMDgzNjUwMmQ5OTVlZDU2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTQ2NjJkMmMwNjcyNDk4MzlkYzRmMTU1ODZlZTM2M2EuYmluZFBvcHVwKHBvcHVwXzM0NWJkZTBlYmUyZjQ4MTg4YjFiNzNhOTQ3OTIwZDA1KQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfZjU2YmJhNDliZTEyNDczMDk3ZWViNmJlYjMzNzE1MTkgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMDQ2MzA3LCAxNzQuOTExNDg0MV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfY2JlMWMxODVlYTk5NGU4YmJmNmIyODZlMDdmNWM1NWEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hYjhlMmIxYjljZGU0ZDdiYjQzZDA4ZWMwMjg0NjRmMiA9ICQoJzxkaXYgaWQ9Imh0bWxfYWI4ZTJiMWI5Y2RlNGQ3YmI0M2QwOGVjMDI4NDY0ZjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIE1lbGxpbmc8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2NiZTFjMTg1ZWE5OTRlOGJiZjZiMjg2ZTA3ZjVjNTVhLnNldENvbnRlbnQoaHRtbF9hYjhlMmIxYjljZGU0ZDdiYjQzZDA4ZWMwMjg0NjRmMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2Y1NmJiYTQ5YmUxMjQ3MzA5N2VlYjZiZWIzMzcxNTE5LmJpbmRQb3B1cChwb3B1cF9jYmUxYzE4NWVhOTk0ZThiYmY2YjI4NmUwN2Y1YzU1YSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2Q3NmNjM2I3MDI0ZjQyNGQ4ZThlN2FkNDIyNjc1M2YyID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMTc4OTI2LCAxNzQuOTYxMTU4OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWY1Mzg2MGM1ZGNmNGQ5YmJhZmRkZTE1ZmY2NTJjOWQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iMzNlMmRiMDRjZTI0YTg1YjRhYmE5NGU2YmI2ZWI2ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfYjMzZTJkYjA0Y2UyNGE4NWI0YWJhOTRlNmJiNmViNmQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFRhaXRhPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xZjUzODYwYzVkY2Y0ZDliYmFmZGRlMTVmZjY1MmM5ZC5zZXRDb250ZW50KGh0bWxfYjMzZTJkYjA0Y2UyNGE4NWI0YWJhOTRlNmJiNmViNmQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9kNzZjYzNiNzAyNGY0MjRkOGU4ZTdhZDQyMjY3NTNmMi5iaW5kUG9wdXAocG9wdXBfMWY1Mzg2MGM1ZGNmNGQ5YmJhZmRkZTE1ZmY2NTJjOWQpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>
