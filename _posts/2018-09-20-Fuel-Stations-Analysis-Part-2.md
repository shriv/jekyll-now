In the previous section, we obtained and plotted locations of Z and BP stations in Wellington, New Zealand. Our goal is to understand the usefulness of the fuel station spatial network. Before this, we can abstract the spatial network into a network. This abstraction allows us to measure and visualise network structure with fairly simple methodology.

Constructing the abstract network requires generation of the connecting edges between fuel stations. Since we're reducing a spatial network, distancea  is a sensible edge metric. In particular, we'd want to know about the distance of the best route between the fuel stations. We build up the abstract network in two steps:
- First, fuel station nodes connected by the best route.
- And a further reduction to fuel station nodes connected by the *distance* of the best route. 


# Introduction to street network analysis
The package, OSMnx (a portmanteau acronym of Open Street Maps, OSM, and NetworkX, nx), is a great package for doing network anlysis with street data. The underlying representation used by this package is a reduction of streets and roads to edges with intersectionsas the vertices (or, nodes). This representation is also known as a 'Primal Graph'. The position of the nodes and the trajectory of the edges are further described with geolocation coordinates. The technical aspects are presented in [this paper](https://arxiv.org/pdf/1611.01890.pdf) by Geoff Boening: the author of OSMnx. 

With the OSMnx package, we can superimpose entities with geolocation on the spatial network. Once we've done this, we can find a path (route) connecting any two nodes. Because of the representation constraints, we don't find the route between the 2 specific entity coordinates (like Google Maps) - instead, we find the path between two nodes closest to the entities. 

The route between the nodes uses the edges (streets and roads) of the spatial network. The route calculation algorithm is an analogue of the [typical shortest path analyses done in network science](https://en.wikipedia.org/wiki/Shortest_path_problem). In a spatial network, the path length can be equated to distance. The base unit of length is metres. 

## Toy Example: route between Z Kilbirnie and Z Vivian St
The following example looks at the distance and route between two Z stations: Z Kilbernie and Z Vivian St. The red line in the figure is the shortest route that connects the two stations. From the street shapes, you can see that the route is wending it's way around Evans Bay and Basin Reserve, before entering the central city street grid. This route has a distance of 4.6 km - a value that corresponds quite closely to that given by [Google Maps](https://bit.ly/2Mvjr0L). 


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_29_0.png)


Text(0.5,1,'Shortest distance between Z Kilbirnie and Z Vivian St is 4577.443 m')


## Get street network for Wellington
 In the toy example, I only downloaded the street network within a 5km radius of Z Kilbirnie. For the main analysis though, we want all the streets and roads within the defined bounding box. The updated figure now shows the route between Z Kilbirnie and Z Vivian St overlayed on all the roads in the Wellington + Lower Hutt bounding box.  


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_31_0.png)

Text(0.5,1,'Shortest distance between Z Kilbirnie and Z Vivian St is 4268.96 m')


# Analysis: Average distance between Z stations in Wellington
This first analysis builds on the toy example to calculate the average distance between any two Z stations. A more academic name for this metric is: *average inter-station separation*. 

The procedure is to first calculate the route and distance between all possible pairs of the 13 Z stations in the region. The following table shows a subset of the results. We see distances from a bunch of Z stations to Z Broadway (in Strathmore). 


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
      <th>distance</th>
      <th>id_from</th>
      <th>from</th>
      <th>id_to</th>
      <th>to</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2822.644</td>
      <td>3120151445</td>
      <td>Z Kilbirnie</td>
      <td>5821475056</td>
      <td>Z Broadway</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1332.762</td>
      <td>5821475059</td>
      <td>Z Miramar</td>
      <td>5821475056</td>
      <td>Z Broadway</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4002.103</td>
      <td>5821475061</td>
      <td>Z Constable Street</td>
      <td>5821475056</td>
      <td>Z Broadway</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5644.063</td>
      <td>5821475058</td>
      <td>Z Taranaki Street</td>
      <td>5821475056</td>
      <td>Z Broadway</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5744.885</td>
      <td>5544110098</td>
      <td>Z Vivian St</td>
      <td>5821475056</td>
      <td>Z Broadway</td>
    </tr>
  </tbody>
</table>
</div>


Once we have 13x13 distances, we can get the closest station from every one of the 13 stations. A plot of the results shows that we have an asymmetric distribution of distances. A significant number clustered around 2km but also some which are more than twice the distance away. The average (both mean and median) inter-station separation is just over 2km 


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_35_0.png)

# Analysis: Number of neighbours for Z stations
The 13x13 table of pairwise distances can be used to analyse the number of neighbours for a Z station within a particular radius. For this analysis, I've recast the data into a network structure. The recasting is useful since we can use some standard network analysis tools available in the networkx package. 

The steps of the recasting are: 
- Filter the 13x13 distances to only include separations less than or equal to 10km. This step would remove the connection between Z Broadway and Z Petone for example. 
- Store the filtered distance matrix as a network data structure. This means:
    - 13 Z stations become nodes
    - Any Z station within 10km of each node becomes a connecting edge
    - The distance value is stored as a weight. With shorter distances have a higher 'weight'

We can visualise the network structure of the simpler, recast data. Some interesting insights include:
- 2 clusters are apparent in the Z station network for Wellington: Wellington City and Lower Hutt. 
- The Wellington City cluster is very tightly connected for the central and southern suburbs. 
- The connectivity of the Wellington City cluster reduces for the northern stations. The table of connections shows that stations in the southern suburbs are connected to two more stations than the northern suburbs and Lower Hutt. 


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_39_0.png)


The explicit connectivity of each Z station is given by a metric called 'degree' in network analysis. The degree distribution is useful for understanding characteristics of structure in larger &/ complex networks. Here, it's simply useful to use the node degree to understand the highly connected / central Z stations. As expected, these stations are the ones in the city centre: Z Harboour City, Z Vivian St, Z Taranaki Street.   

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
      <th>station</th>
      <th>degree</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Z Harbour City</td>
      <td>8</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Z Vivian St</td>
      <td>8</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Z Taranaki Street</td>
      <td>8</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Z Constable Street</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Z Miramar</td>
      <td>6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Z Broadway</td>
      <td>6</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Z Kilbirnie</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Z Crofton Downs</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Z Petone</td>
      <td>5</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Z Johnsonville</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Z Hutt Road</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Z High Street</td>
      <td>4</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Z VIC Corner</td>
      <td>4</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Z Seaview</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>


The average degree / connectivity for the Wellington City Z stations is much higher than Lower Hutt. The typical Z station in Wellington City is connected to 2 more Z stations than a typicsl Z station in Lower Hutt.   

    Z stations in Wellington region have an average of 5.71 neighbours
    Z stations in Wellington City have an average of 6.5 neighbours
    Z stations in Lower Hutt have an average of 4.67 neighbours


<a id='Competitor-Analysis'></a>
# Competitor Analysis: Number of neighbours for BP stations
The same network analysis can be done for a competitor. I chose BP, since it *seems* to have a similar coverage to Z in the Wellington region - extending in an arc from Wellington City to the Northern Suburbs to Hutt Valley. 


<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVM9ZmFsc2U7IExfTk9fVE9VQ0g9ZmFsc2U7IExfRElTQUJMRV8zRD1mYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgPHN0eWxlPiNtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMgewogICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgdG9wOiAwLjAlOwogICAgICAgIH0KICAgIDwvc3R5bGU+CjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjIiA+PC9kaXY+CjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgIAogICAgICAgIHZhciBib3VuZHMgPSBudWxsOwogICAgCgogICAgdmFyIG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyA9IEwubWFwKAogICAgICAgICdtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMnLCB7CiAgICAgICAgY2VudGVyOiBbLTQxLjI5LCAxNzQuOF0sCiAgICAgICAgem9vbTogMTEsCiAgICAgICAgbWF4Qm91bmRzOiBib3VuZHMsCiAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3LAogICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgIH0pOwoKICAgIAogICAgCiAgICB2YXIgdGlsZV9sYXllcl83ZTU1YmE1Y2QyMWU0YWQwOTVkYTdkOTBkYWZiZDY3NSA9IEwudGlsZUxheWVyKAogICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgewogICAgICAgICJhdHRyaWJ1dGlvbiI6IG51bGwsIAogICAgICAgICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgCiAgICAgICAgIm1heE5hdGl2ZVpvb20iOiAxOCwgCiAgICAgICAgIm1heFpvb20iOiAxOCwgCiAgICAgICAgIm1pblpvb20iOiAwLCAKICAgICAgICAibm9XcmFwIjogZmFsc2UsIAogICAgICAgICJzdWJkb21haW5zIjogImFiYyIKfSkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgIAogICAgICAgIHZhciBtYXJrZXJfMTE1NGM2MTc5ODgyNDM4YmE1MjBmMDkxMWIyNmU0YzEgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMTk2NzE5LCAxNzQuNzc1NjQ2OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGRmY2UxNTUzODY5NGQxNWJjYWI1ZmY1MzMzNzYyNTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85YjFmMTc1ZjU4ZWU0MjNhOGFlZjk1OTlhZGMwZGU0NSA9ICQoJzxkaXYgaWQ9Imh0bWxfOWIxZjE3NWY1OGVlNDIzYThhZWY5NTk5YWRjMGRlNDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEJlcmhhbXBvcmU8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBkZmNlMTU1Mzg2OTRkMTViY2FiNWZmNTMzMzc2MjU4LnNldENvbnRlbnQoaHRtbF85YjFmMTc1ZjU4ZWU0MjNhOGFlZjk1OTlhZGMwZGU0NSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzExNTRjNjE3OTg4MjQzOGJhNTIwZjA5MTFiMjZlNGMxLmJpbmRQb3B1cChwb3B1cF8wZGZjZTE1NTM4Njk0ZDE1YmNhYjVmZjUzMzM3NjI1OCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzM5ODA2NGU1NTBlNjQ0MzE5ZjQxZjJjNjNhYTAwNDExID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMzA2NzMxMSwgMTc0Ljc2MzM2OTVdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MzOTA4ZWExNjc2YzQzMmM4ZjA0NTZhNjMxZGNhYTdmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2U2ZDAyNTY2ZTEyNDU4NjhiNTZlYTAwZWRlZmRmYzAgPSAkKCc8ZGl2IGlkPSJodG1sXzdlNmQwMjU2NmUxMjQ1ODY4YjU2ZWEwMGVkZWZkZmMwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBCcm9va2x5bjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzM5MDhlYTE2NzZjNDMyYzhmMDQ1NmE2MzFkY2FhN2Yuc2V0Q29udGVudChodG1sXzdlNmQwMjU2NmUxMjQ1ODY4YjU2ZWEwMGVkZWZkZmMwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMzk4MDY0ZTU1MGU2NDQzMTlmNDFmMmM2M2FhMDA0MTEuYmluZFBvcHVwKHBvcHVwX2MzOTA4ZWExNjc2YzQzMmM4ZjA0NTZhNjMxZGNhYTdmKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfODZjYmQxYjU3MzliNGI2NWI2OTk3YzhlNjA1NjBlYTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4zMDMwMzYxLCAxNzQuNzc5NjA1Ml0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTQ0NTA2MGRmZmMxNGE0ZWJlODExNWIzOTEyMTIxMWUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yODZkYzNkOTJjYzY0ZjIyOTY0Y2E0M2VjNjRkOTQyYSA9ICQoJzxkaXYgaWQ9Imh0bWxfMjg2ZGMzZDkyY2M2NGYyMjk2NGNhNDNlYzY0ZDk0MmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEFkZWxhaWRlIFJvYWQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzU0NDUwNjBkZmZjMTRhNGViZTgxMTViMzkxMjEyMTFlLnNldENvbnRlbnQoaHRtbF8yODZkYzNkOTJjYzY0ZjIyOTY0Y2E0M2VjNjRkOTQyYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzg2Y2JkMWI1NzM5YjRiNjViNjk5N2M4ZTYwNTYwZWExLmJpbmRQb3B1cChwb3B1cF81NDQ1MDYwZGZmYzE0YTRlYmU4MTE1YjM5MTIxMjExZSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2Y3ZjUwNjdhNjEzNTQzMDhhNDE0M2E5ODA0M2I2ZmY3ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjkwOTUxLCAxNzQuNzgwMjEyNl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNmVjYjhhMzNhMDZiNGVhNzk0MTcyYjgzZWY4ODVjZjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MzcxZTlmMDQ4MzM0Njc4OGY3M2Y1ODUzYzM0M2NkZCA9ICQoJzxkaXYgaWQ9Imh0bWxfNzM3MWU5ZjA0ODMzNDY3ODhmNzNmNTg1M2MzNDNjZGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFJvYWRtYXN0ZXI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzZlY2I4YTMzYTA2YjRlYTc5NDE3MmI4M2VmODg1Y2YwLnNldENvbnRlbnQoaHRtbF83MzcxZTlmMDQ4MzM0Njc4OGY3M2Y1ODUzYzM0M2NkZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2Y3ZjUwNjdhNjEzNTQzMDhhNDE0M2E5ODA0M2I2ZmY3LmJpbmRQb3B1cChwb3B1cF82ZWNiOGEzM2EwNmI0ZWE3OTQxNzJiODNlZjg4NWNmMCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2MyMzhmYzNkYTM5NzQ0YjY4NWRkYjViMWJhY2IyOTYzID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjg0ODU4OCwgMTc0LjczNjM5NjVdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzlkNjZkYjE1YWFiYzRmZTE5MjY2Y2Y1YWY3MzQ0YTkyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzEyMGZkMTQ2ZTg5NDcxNjk5NjU4Y2ZkZGM4MDlhYmMgPSAkKCc8ZGl2IGlkPSJodG1sX2MxMjBmZDE0NmU4OTQ3MTY5OTY1OGNmZGRjODA5YWJjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBLYXJvcmk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzlkNjZkYjE1YWFiYzRmZTE5MjY2Y2Y1YWY3MzQ0YTkyLnNldENvbnRlbnQoaHRtbF9jMTIwZmQxNDZlODk0NzE2OTk2NThjZmRkYzgwOWFiYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2MyMzhmYzNkYTM5NzQ0YjY4NWRkYjViMWJhY2IyOTYzLmJpbmRQb3B1cChwb3B1cF85ZDY2ZGIxNWFhYmM0ZmUxOTI2NmNmNWFmNzM0NGE5MikKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzUyYjU3MTU3ZWUyNzQ2Y2Q5MjNlOTcxNWUzMjc3MjM5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjYyNjcyOCwgMTc0Ljk0NjE0MTFdLAogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hZmEwMjFjOGEwMWQ0MGM5YjgzMzQ0NzFiM2ZkMTJmYyk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzU2YjMxZjU1YjVmMjRmM2I5ZjgwM2MwYzczZGMzNjIyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnCiAgICAgICAgICAgIAogICAgICAgICAgICB9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDJhNzdmMTkzODlhNDNiYjk3ZjAxM2Q1YTJjOGJkNGYgPSAkKCc8ZGl2IGlkPSJodG1sX2QyYTc3ZjE5Mzg5YTQzYmI5N2YwMTNkNWEyYzhiZDRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CUCBXYWludWlvbWF0YTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTZiMzFmNTViNWYyNGYzYjlmODAzYzBjNzNkYzM2MjIuc2V0Q29udGVudChodG1sX2QyYTc3ZjE5Mzg5YTQzYmI5N2YwMTNkNWEyYzhiZDRmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTJiNTcxNTdlZTI3NDZjZDkyM2U5NzE1ZTMyNzcyMzkuYmluZFBvcHVwKHBvcHVwXzU2YjMxZjU1YjVmMjRmM2I5ZjgwM2MwYzczZGMzNjIyKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfYzc5NDVjZDk0ODU0NGEyNWE1Y2VkYzRhODlkMjkzMzIgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yNDEyMTU5LCAxNzQuOTA4MTc4OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjcxM2MyYjhlOWUyNDc5NTllZjE2NzFhNTNiYjI1Y2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jY2QyYzlmZTdlMDM0ZmNmYmZmYWJjZGRjMzBmMWZjNiA9ICQoJzxkaXYgaWQ9Imh0bWxfY2NkMmM5ZmU3ZTAzNGZjZmJmZmFiY2RkYzMwZjFmYzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFNlYXZpZXcgVHJ1Y2tzdG9wPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mNzEzYzJiOGU5ZTI0Nzk1OWVmMTY3MWE1M2JiMjVjYi5zZXRDb250ZW50KGh0bWxfY2NkMmM5ZmU3ZTAzNGZjZmJmZmFiY2RkYzMwZjFmYzYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9jNzk0NWNkOTQ4NTQ0YTI1YTVjZWRjNGE4OWQyOTMzMi5iaW5kUG9wdXAocG9wdXBfZjcxM2MyYjhlOWUyNDc5NTllZjE2NzFhNTNiYjI1Y2IpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl81NmUwNjczMTgzMGY0ZThhYWIzMWViNjQ2MTc5MzM0ZSA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIzMjA4ODcsIDE3NC44Mzc0MDYyXSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lNmYzNjg5OWUwNzA0Njk0YjFiZmU2YWYzOGE5NzM4OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VlNDQ4YjVlZDY2ZTRjNmI5NWQxYjFhNjJkMTcyOTFjID0gJCgnPGRpdiBpZD0iaHRtbF9lZTQ0OGI1ZWQ2NmU0YzZiOTVkMWIxYTYyZDE3MjkxYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgSHV0dCBSb2FkPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lNmYzNjg5OWUwNzA0Njk0YjFiZmU2YWYzOGE5NzM4OC5zZXRDb250ZW50KGh0bWxfZWU0NDhiNWVkNjZlNGM2Yjk1ZDFiMWE2MmQxNzI5MWMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl81NmUwNjczMTgzMGY0ZThhYWIzMWViNjQ2MTc5MzM0ZS5iaW5kUG9wdXAocG9wdXBfZTZmMzY4OTllMDcwNDY5NGIxYmZlNmFmMzhhOTczODgpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl85NDdjOTc4Y2YxY2E0OGVkOTJkYTNkZmQwOGVkYjNjZiA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIyOTc1NTksIDE3NC44MTc2MjA0XSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yMmRjNzIxMWFmZjQ0MzQxYTdmOWZiM2E2MmM5ZTAyMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUxYzQ4ODhiYmM4NjRlNzU5ODc3OGQ0MDEzMjJlMDYzID0gJCgnPGRpdiBpZD0iaHRtbF81MWM0ODg4YmJjODY0ZTc1OTg3NzhkNDAxMzIyZTA2MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgTmV3bGFuZHM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzIyZGM3MjExYWZmNDQzNDFhN2Y5ZmIzYTYyYzllMDIwLnNldENvbnRlbnQoaHRtbF81MWM0ODg4YmJjODY0ZTc1OTg3NzhkNDAxMzIyZTA2Myk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzk0N2M5NzhjZjFjYTQ4ZWQ5MmRhM2RmZDA4ZWRiM2NmLmJpbmRQb3B1cChwb3B1cF8yMmRjNzIxMWFmZjQ0MzQxYTdmOWZiM2E2MmM5ZTAyMCkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyXzQzNzcwY2Q0NGJjYjQ5MzBhNjE3ODVjMTU1YTdhM2VmID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMjI1ODU3LCAxNzQuODA3NDkyMl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMTAzOWQzNmFiYmRiNGNjYzg4ODM5OTE1NjQ4ZDJlNmQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85ZDMzOWJkZjVlNDQ0ZTc1YjBhNjVlMmVjYWJhZDM2ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfOWQzMzliZGY1ZTQ0NGU3NWIwYTY1ZTJlY2FiYWQzNmYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIEpvaG5zb252aWxsZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTAzOWQzNmFiYmRiNGNjYzg4ODM5OTE1NjQ4ZDJlNmQuc2V0Q29udGVudChodG1sXzlkMzM5YmRmNWU0NDRlNzViMGE2NWUyZWNhYmFkMzZmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNDM3NzBjZDQ0YmNiNDkzMGE2MTc4NWMxNTVhN2EzZWYuYmluZFBvcHVwKHBvcHVwXzEwMzlkMzZhYmJkYjRjY2M4ODgzOTkxNTY0OGQyZTZkKQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfOTQ2NGRmMTc1OTkwNDIyYjkxM2MwNDQyOGZjODJlYTMgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMjI4Mjg1LCAxNzQuOTExOTYyMl0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNGE0ODg0NGFkNjY5NGQ4NWI3ZjQ1MjY2YWRmZjVmM2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lYjFhNDc5Nzk0MjY0NThjYTEyODgxNjIyMGE2YmM2OCA9ICQoJzxkaXYgaWQ9Imh0bWxfZWIxYTQ3OTc5NDI2NDU4Y2ExMjg4MTYyMjBhNmJjNjgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFdhaXdoZXR1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80YTQ4ODQ0YWQ2Njk0ZDg1YjdmNDUyNjZhZGZmNWYzYi5zZXRDb250ZW50KGh0bWxfZWIxYTQ3OTc5NDI2NDU4Y2ExMjg4MTYyMjBhNmJjNjgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl85NDY0ZGYxNzU5OTA0MjJiOTEzYzA0NDI4ZmM4MmVhMy5iaW5kUG9wdXAocG9wdXBfNGE0ODg0NGFkNjY5NGQ4NWI3ZjQ1MjY2YWRmZjVmM2IpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgdmFyIG1hcmtlcl81NDY2MmQyYzA2NzI0OTgzOWRjNGYxNTU4NmVlMzYzYSA9IEwubWFya2VyKAogICAgICAgICAgICBbLTQxLjIxMjYwNzEsIDE3NC44OTM2NTQ4XSwKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYWZhMDIxYzhhMDFkNDBjOWI4MzM0NDcxYjNmZDEyZmMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zNDViZGUwZWJlMmY0ODE4OGIxYjczYTk0NzkyMGQwNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJwogICAgICAgICAgICAKICAgICAgICAgICAgfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE4YjU4MjBmZTdlYjQ2MzZiMDgzNjUwMmQ5OTVlZDU2ID0gJCgnPGRpdiBpZD0iaHRtbF8xOGI1ODIwZmU3ZWI0NjM2YjA4MzY1MDJkOTk1ZWQ1NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QlAgUmFpbHdheTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMzQ1YmRlMGViZTJmNDgxODhiMWI3M2E5NDc5MjBkMDUuc2V0Q29udGVudChodG1sXzE4YjU4MjBmZTdlYjQ2MzZiMDgzNjUwMmQ5OTVlZDU2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTQ2NjJkMmMwNjcyNDk4MzlkYzRmMTU1ODZlZTM2M2EuYmluZFBvcHVwKHBvcHVwXzM0NWJkZTBlYmUyZjQ4MTg4YjFiNzNhOTQ3OTIwZDA1KQogICAgICAgICAgICA7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgIHZhciBtYXJrZXJfZjU2YmJhNDliZTEyNDczMDk3ZWViNmJlYjMzNzE1MTkgPSBMLm1hcmtlcigKICAgICAgICAgICAgWy00MS4yMDQ2MzA3LCAxNzQuOTExNDg0MV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfY2JlMWMxODVlYTk5NGU4YmJmNmIyODZlMDdmNWM1NWEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hYjhlMmIxYjljZGU0ZDdiYjQzZDA4ZWMwMjg0NjRmMiA9ICQoJzxkaXYgaWQ9Imh0bWxfYWI4ZTJiMWI5Y2RlNGQ3YmI0M2QwOGVjMDI4NDY0ZjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIE1lbGxpbmc8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2NiZTFjMTg1ZWE5OTRlOGJiZjZiMjg2ZTA3ZjVjNTVhLnNldENvbnRlbnQoaHRtbF9hYjhlMmIxYjljZGU0ZDdiYjQzZDA4ZWMwMjg0NjRmMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2Y1NmJiYTQ5YmUxMjQ3MzA5N2VlYjZiZWIzMzcxNTE5LmJpbmRQb3B1cChwb3B1cF9jYmUxYzE4NWVhOTk0ZThiYmY2YjI4NmUwN2Y1YzU1YSkKICAgICAgICAgICAgOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICB2YXIgbWFya2VyX2Q3NmNjM2I3MDI0ZjQyNGQ4ZThlN2FkNDIyNjc1M2YyID0gTC5tYXJrZXIoCiAgICAgICAgICAgIFstNDEuMTc4OTI2LCAxNzQuOTYxMTU4OV0sCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2FmYTAyMWM4YTAxZDQwYzliODMzNDQ3MWIzZmQxMmZjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWY1Mzg2MGM1ZGNmNGQ5YmJhZmRkZTE1ZmY2NTJjOWQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCcKICAgICAgICAgICAgCiAgICAgICAgICAgIH0pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iMzNlMmRiMDRjZTI0YTg1YjRhYmE5NGU2YmI2ZWI2ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfYjMzZTJkYjA0Y2UyNGE4NWI0YWJhOTRlNmJiNmViNmQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJQIFRhaXRhPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xZjUzODYwYzVkY2Y0ZDliYmFmZGRlMTVmZjY1MmM5ZC5zZXRDb250ZW50KGh0bWxfYjMzZTJkYjA0Y2UyNGE4NWI0YWJhOTRlNmJiNmViNmQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9kNzZjYzNiNzAyNGY0MjRkOGU4ZTdhZDQyMjY3NTNmMi5iaW5kUG9wdXAocG9wdXBfMWY1Mzg2MGM1ZGNmNGQ5YmJhZmRkZTE1ZmY2NTJjOWQpCiAgICAgICAgICAgIDsKCiAgICAgICAgICAgIAogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



The interconnectivity network for BP shares some similar characteristics to the Z network but also has some obvious differences. 
- Wellington City and Lower Hutt clusters persist.
- The northern suburbs are a little better connected. 
- The Wellington cluster is not as well connected as the Z station network. 
- The Lower Hutt cluster for BP is much better connected than Z. BP also has 2 more stations in Lower Hutt compared to Z. 

All these points indicate that while Z and BP cover similar areas of Wellington, *Z is better represented in Wellington City while BP dominates in Lower Hutt*. It would be very interesting to see if revenue per station is signficantly different for a Z station in Wellington City vs. Lower Hutt.  


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_48_0.png)


Because the Z station network was a loosely connected network with two strongly connected clusters, we could calculate the average degree per cluster, with little loss of accuracy. The BP network doesn't have the same structure - BP stations are reasonably well connected throughout the Wellington region network. 

# Competitor Analysis: Average distance between stations (Z vs. BP)

The physical coverage of Z vs. BP stations using the inter-station separation continues to show the asymmetric distribution of separations. The key difference is that some BP stations are *much* better connected than others. The effect is more noticeable than for Z.

From this comparison, we can say that Z stations are better spread in the Wellington region compared to BP. We need to exercise some caution however; with only ~13 stations, we don't have much sample size. If we do a more complete analysis for Z, we can get robust statistics by running a hierarchical model for the average inter-station separation across the different types of regions. Until then, we just have to be mindful of how strongly we present this message. 


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel%20Stations%20Analysis_51_0.png)


    Average (mean) inter-station distances:
    - Z stations in Wellington are 2.412 km apart on average
    - BP stations in Wellington are 3.125 km apart on average


# Analysis: Nearest neighbours in joint Z- BP fuel station network

A key characteristic of good coverage is location in relation to other entities - especially competitors. A franchise should ideally be placed close to its own rather than near a competitor. A simple set of comparative analyses that explore the type pf nearest neighbour include:
- Seeing whether Z stations neighbour each other or a competitor
- Which Z stations are in a zone of poaching risk - i.e. their customers might go to a nearby competitor. 

For this analysis, we need to generate the shortest distances between all station pairs for *both* Z and BP stations together. The computation is not fast unfortunately so we definitely need to manage this calculation better for a larger dataset. Also, N.B. there is an implicit redundancy in the numbers cited above: some station pairs are each others nearest neighbours. 


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
      <th>from</th>
      <th>to</th>
      <th>distance</th>
      <th>from_brand</th>
      <th>to_brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19</th>
      <td>BP Melling</td>
      <td>Z VIC Corner</td>
      <td>158.905</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Z Johnsonville</td>
      <td>BP Johnsonville</td>
      <td>165.593</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BP Johnsonville</td>
      <td>Z Johnsonville</td>
      <td>165.593</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Z VIC Corner</td>
      <td>BP Melling</td>
      <td>185.207</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Z Vivian St</td>
      <td>Z Taranaki Street</td>
      <td>436.455</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
  </tbody>
</table>
</div>


> Out of 28 Z and BP stations, 19 are closest to a competitor and 9 are next to one of their own
> Of the 9 stations next to their own, 8 are from Z
> Of the 19 stations next to a competitor, 10 are from Z


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
      <th>from</th>
      <th>to</th>
      <th>distance</th>
      <th>from_brand</th>
      <th>to_brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>Z Vivian St</td>
      <td>Z Taranaki Street</td>
      <td>436.455</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Z Taranaki Street</td>
      <td>Z Vivian St</td>
      <td>438.123</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Z Miramar</td>
      <td>Z Broadway</td>
      <td>1332.762</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Z Broadway</td>
      <td>Z Miramar</td>
      <td>1332.762</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Z Kilbirnie</td>
      <td>Z Miramar</td>
      <td>2111.346</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Z Petone</td>
      <td>Z Hutt Road</td>
      <td>2227.071</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Z High Street</td>
      <td>Z VIC Corner</td>
      <td>2356.524</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Z Crofton Downs</td>
      <td>Z Harbour City</td>
      <td>5270.263</td>
      <td>Z</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>25</th>
      <td>BP Wainuiomata</td>
      <td>BP Waiwhetu</td>
      <td>6335.055</td>
      <td>BP</td>
      <td>BP</td>
    </tr>
  </tbody>
</table>
</div>



Z stations with a BP station within the *average station-station separation distance* are at risk of having their users poached by the competitor. The list below shows that key poaching zones are: Johnsonville, Western / Central Hutt, Seaview, central Wellington and south-central Wellington (Newtown, Berhampore).  


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
      <th>from</th>
      <th>to</th>
      <th>distance</th>
      <th>from_brand</th>
      <th>to_brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19</th>
      <td>BP Melling</td>
      <td>Z VIC Corner</td>
      <td>158.905</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Z Johnsonville</td>
      <td>BP Johnsonville</td>
      <td>165.593</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BP Johnsonville</td>
      <td>Z Johnsonville</td>
      <td>165.593</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Z VIC Corner</td>
      <td>BP Melling</td>
      <td>185.207</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>6</th>
      <td>BP Roadmaster</td>
      <td>Z Taranaki Street</td>
      <td>729.384</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BP Seaview Truckstop</td>
      <td>Z Seaview</td>
      <td>731.884</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Z Seaview</td>
      <td>BP Seaview Truckstop</td>
      <td>748.029</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>5</th>
      <td>BP Adelaide Road</td>
      <td>Z Taranaki Street</td>
      <td>1003.535</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Z Harbour City</td>
      <td>BP Roadmaster</td>
      <td>1162.016</td>
      <td>Z</td>
      <td>BP</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BP Berhampore</td>
      <td>Z Constable Street</td>
      <td>1351.750</td>
      <td>BP</td>
      <td>Z</td>
    </tr>
  </tbody>
</table>
</div>
