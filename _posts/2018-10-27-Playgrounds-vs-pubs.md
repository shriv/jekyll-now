This post applies accessibility analysis to compare two types of social amenities: council parks and alcohol vendors - a wider scope than the cheeky alliterative title would suggest. While the accessibility concept is unchanged from the [fuel station analysis](https://shriv.github.io/Fuel-Stations-Analysis-Part-3/), the post shows how we can bring together different types (and sources) of spatial data for a richer, and ultimately more insightful analysis.

This post covers:
- Getting and processing OpenStreetMap (OSM) ways and nodes into point entities.
- Converting polygon data into points.
- Bringing together the two different spatial sources for comparing accessibility.
- Manipulating the pandana data structure to plot accessibilities within regions of interest.

If bits of the above list don't make sense, feel free to visit the previous blog post for an [introduction to OSM](https://shriv.github.io/Fuel-Stations-Analysis-Part-1/) and [accessibility analysis](https://shriv.github.io/Fuel-Stations-Analysis-Part-3/). As usual, the code for this post is up as a [Jupyter notebook on my github](https://github.com/shriv/playgrounds-pubs/blob/master/Playgrounds%20vs%20Pubs.ipynb).


# Getting alcohol vendors from OSM
As highlighted in a [previous post](https://shriv.github.io/Fuel-Stations-Analysis-Part-1/), getting data from OSM is quite easy with the Overpass API. For this analysis, 'shop', 'amenity' and 'building' entities with the tags 'alcohol', 'pub', 'bar', 'beverages', 'biergarten', 'wine' and 'supermarket' were extracted from OSM. The key difference to the fuel stations analysis is that we have to extend the query to deal with both nodes _and_ ways. For a complete dataset of alcohol vendors, we need two queries since the results are different for nodes and ways.

Following the data extraction we have to:
- Process ways as polygons and then reduce to POIs
- Label nodes as POIs
- Join the two datasets together

There is a reasonable probability that we have a small number of duplicates - where the same place has been annotated as both a way and a node. However, for the first pass of the analysis, I'm going to assume that these are negligible. De-duplication will be a part of a more polished, future analysis.


## Nodes
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
      <th>amenity</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4522166458</td>
      <td>-41.328972</td>
      <td>174.811298</td>
      <td>Cook Strait Bar</td>
      <td>bar</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>623879839</td>
      <td>-41.325184</td>
      <td>174.820872</td>
      <td>The Strathmore Local</td>
      <td>pub</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>625080280</td>
      <td>-41.319506</td>
      <td>174.794358</td>
      <td>Bay 66</td>
      <td>pub</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4362160389</td>
      <td>-41.318292</td>
      <td>174.794757</td>
      <td>No Name</td>
      <td>No Name</td>
      <td>node</td>
    </tr>
    <tr>
      <th>4</th>
      <td>627273501</td>
      <td>-41.318117</td>
      <td>174.794450</td>
      <td>Kilbirnie Tavern</td>
      <td>pub</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>



## Get and process ways
Processing ways for accessibility analysis means we need to find a way of reducing polygons to points.

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
      <th>amenity</th>
      <th>type</th>
      <th>nodes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>26509771</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>New World</td>
      <td>NaN</td>
      <td>way</td>
      <td>[290565312, 290565316, 2990208452, 2990208451,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>49396700</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Countdown</td>
      <td>NaN</td>
      <td>way</td>
      <td>[627273504, 4699896634, 627273505, 4199712656,...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>62153738</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mac's Brewery</td>
      <td>pub</td>
      <td>way</td>
      <td>[775428527, 775428528, 775428657, 775428658, 2...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>62154227</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Countdown Johnsonville</td>
      <td>NaN</td>
      <td>way</td>
      <td>[1439843310, 1439843337, 1439843339, 143984333...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>133129214</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Thistle Inn</td>
      <td>bar</td>
      <td>way</td>
      <td>[1464807182, 1464807184, 1464807179, 146480718...</td>
    </tr>
  </tbody>
</table>
</div>



![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/map_alcohol.html)

# Quality of alcohol vendor data
The paper by [Bright et al.](https://www.sciencedirect.com/science/article/pii/S1353829217305804) gives an excellent overview of why alcohol research is important - in terms of spatial availability. It also evaluates the use of alcohol vendor data from Open Street Map. Though the analysis was carried out for the UK, we can extrapolate the general principle that data is unlikely to be complete for NZ. Possibly even more so since we're a smaller country, and OSM has a much greater set of contributers in the UK - owing to the fact that OSM began in the UK.

## Alternatives: NZLII
An alternative source of alcohol data to OSM is the [New Zealand Legal Information Institute's](http://www.nzlii.org/) database on alcohol licence decsions. The site for Wellington is [here](http://www.nzlii.org/nz/cases/NZDLCWN/). The licence [requests for the current year](http://www.nzlii.org/nz/cases/NZDLCWN/2018/) can be examined in detail. Unfortunately, the decisions are uploaded as pdfs.

While text can be extracted from the pdfs, the retreival of information requires a few components:
- Web scraping with bs4 to systematically download all the pdfs of decisions for a given year.
- Loading the pdf as an object with PyPDF2.
- Searching the text with well-constructed regular expressions to get the name of venue and the decision.
- Matching the licence venues to a geolocation. Adding in missing venues to OSM.

I think this can actually be done. I'd need some help in crafting the regular expression and, the project will take some time - which menas I need to decide how and why it would be worth scraping the database.


## Alternatives: Healthspace
A Google search led me to this page which [aggregates alcohol-related data and research for NZ](https://www.alcohol.org.nz/resources-research/facts-and-statistics/where-to-find-other-alcohol-statistics). From there, I navigated to Healthspace, which makes available [Statistical Area 2 level alcohol licence data](http://healthspace.ac.nz/resource/view?resourceId=37). The data download portal [here](http://healthspace.ac.nz/explorer/resources/listbytheme) as xlsx tables. The interface is clunky unfortunately and the latest data is from 2016. Nonetheless, if we want SA2 or larger area unit aggregates, this data is great. I will make a request to see if they have the licence information available at the individual outlet level.

# Get parks data
We have two sources of park data available from WCC:
- [WCC parks and reserves](https://data-wcc.opendata.arcgis.com/datasets/581d698fd2614a4c8f860c8007e4e104_0)
- [WCC playgrounds](https://data-wcc.opendata.arcgis.com/datasets/c3b0ae6ee9d44a7786b0990e6ea39e5d_0)

As of 18 Sept, I've only done the analysis on the polygon data of parks and reserves. Repeating the analysis with playgrounds as the focus will be on how family friendly the region is.


## Sample points from a polygon

<iframe src={{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/map_parks.html
style="width: 100%; height: 450px;"></iframe>


# Accessibility analysis

## Calculating accessibility
Here, we consider accessibility as the driving distance in meters from each grid point (also referred to as nodes) to the nearest POIS: a fuel station. To do a visual acessibility analysis we need to:
- Break up the map into grid of points (I)
- Calculate the distance from each point to the nth nearest POIS (II)
- Visualise distance as a heatmap (III)

All the above steps are carried out by the Python package Pandana. Of the above steps, I has a few sub-steps. These are:
- Download OSM data within the specified bounding box
- Convert map to point grid. Remember, this is easy since all OSM streets and roads are *ways* which are simply a collection of nodes / points.
- Store points data in a convenient data structure: a Pandas dataframe
- Filter out poorly connected points



![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_27_0.png)

![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_28_0.png)


## Differential accessbility: playgrounds vs. alcohol


![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_30_0.png)


![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_33_0.png)

![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_34_0.png)



## Accessibility statistics

![]({{ site.baseurl }}/images/2018-10-27-Playgrounds-vs-pubs/Playgrounds%20vs%20Pubs_37_1.png)
