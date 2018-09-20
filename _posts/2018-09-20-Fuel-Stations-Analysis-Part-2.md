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


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_29_0.png)


Text(0.5,1,'Shortest distance between Z Kilbirnie and Z Vivian St is 4577.443 m')


## Get street network for Wellington
 In the toy example, I only downloaded the street network within a 5km radius of Z Kilbirnie. For the main analysis though, we want all the streets and roads within the defined bounding box. The updated figure now shows the route between Z Kilbirnie and Z Vivian St overlayed on all the roads in the Wellington + Lower Hutt bounding box.  


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_31_0.png)

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


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_35_0.png)

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


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_39_0.png)


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




The interconnectivity network for BP shares some similar characteristics to the Z network but also has some obvious differences. 
- Wellington City and Lower Hutt clusters persist.
- The northern suburbs are a little better connected. 
- The Wellington cluster is not as well connected as the Z station network. 
- The Lower Hutt cluster for BP is much better connected than Z. BP also has 2 more stations in Lower Hutt compared to Z. 

All these points indicate that while Z and BP cover similar areas of Wellington, *Z is better represented in Wellington City while BP dominates in Lower Hutt*. It would be very interesting to see if revenue per station is signficantly different for a Z station in Wellington City vs. Lower Hutt.  


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_48_0.png)


Because the Z station network was a loosely connected network with two strongly connected clusters, we could calculate the average degree per cluster, with little loss of accuracy. The BP network doesn't have the same structure - BP stations are reasonably well connected throughout the Wellington region network. 

# Competitor Analysis: Average distance between stations (Z vs. BP)

The physical coverage of Z vs. BP stations using the inter-station separation continues to show the asymmetric distribution of separations. The key difference is that some BP stations are *much* better connected than others. The effect is more noticeable than for Z.

From this comparison, we can say that Z stations are better spread in the Wellington region compared to BP. We need to exercise some caution however; with only ~13 stations, we don't have much sample size. If we do a more complete analysis for Z, we can get robust statistics by running a hierarchical model for the average inter-station separation across the different types of regions. Until then, we just have to be mindful of how strongly we present this message. 


![png]({{ site.baseurl }}/images/2018-09-20-Fuel-Stations-Analysis/Fuel Stations Analysis_51_0.png)


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
