---
layout: post
title: Understanding walkability in Wellington with accessibility analyses
---

In his recent book, _Order without Design_, Alain Bertaud beautifully lays out the value proposition of urban life:

<blockquote>
<ul style='font-size: 90%;'>
<li> A commute short enough that one has time for leisure activities.</li>
<li> An open job market that allows one to change jobs and - through trial and error - find a job that best suits. </li>
<li> A residence from which access to social life or nature is quick and easy.</li>
</ul>
</blockquote>

<p style='font-size: 90%; text-align: right; font-style:italic;'>
  - Order without Design, Alain Bertaud (p 19)
</p>
<br>


As a life-long urbanite with stints around the world, I have given thought to each of these points when evaluating a place to live - from the city itself to a suburb / area. I even moved back to New Zealand after a few years in the UK solely for #2 and #3. As I was looking for rental property, #1 and #3 were primary on my mind.

The above three points are typical core values for _individuals_. For individuals in a family, #3 would grow to include other amenities like schools, medical services, retail etc. A comprehensive evaluation of amenities can be found in metrics like the NDAI (Neighbourhood Destination Accessibility Index).

## Accessing amenities
As urbanites, we can access an amenity with several modes of passive or active transport. Passive transport is usually _motorised_ transport like a car, bus, train or hired vehicles (taxi or Uber). Active transport is a _physical activity_ - like walking, cycling, running, skateboarding or even scooting! Enabling and encouraging active transport modes has become particularly relevant in the current climes.

<blockquote>
<p style='font-size: 90%;'>
Reducing car reliance and encouraging more transport-related physical activity are now recognised as beneficial objectives from health, social and environmental perspectives. Evidence is accumulating that a number of built environment attributes are associated with the likelihood of residents using active transport.
</p>
</blockquote>

<p style='font-size: 90%; text-align: right; font-style:italic;'>
  – Measuring neighbourhood walkability in NZ cities</p>
<br>

The above quote from a [research paper published by Knowledge Auckland](http://knowledgeauckland.org.nz/assets/publications/Measuring_Neighbourhood_Walkability_in_New_Zealand_Cities.pdf) succinctly summarises the connection between transport, physical activity and the built urban environment. Urban attributes are critical for understanding if people will substitute automobile transport (particularly the car) for any choice of active transport.

<blockquote>
<p style='font-size: 90%;'>
Valid and reliable measures of these urban attributes are critical for improving our understanding of the relationship between the built environment and transport mode use.
</p>
</blockquote>

<p style='font-size: 90%; text-align: right; font-style:italic;'>
  – Measuring neighbourhood walkability in NZ cities
</p>


## Objective measures
In this series, the analyses will be constrained to walking - rather than a comprehensive view of active transport. The combined impact of urban attributes that enable walking can be measured objectively with _Walkability_ metrics. [WalkScore](www.walkscore.com) is _an_ implementation of walkability. The Walkability Index, described in _Measuring Neighbourhood Walkability in New Zealand Cities_, is another.

Walkability typically presents a comprehensive picture. But at its core is the predominant question that encapsulates opportunity cost for any transport mode:
 > How long will it take me?

While we all consider itinerary-specific travel times, a general view of travel times to amenities can be quantified and visualised with [accessibility heatmaps](https://towardsdatascience.com/measuring-pedestrian-accessibility-97900f9e4d56).


## Probing walkability in Wellington with accessibility
This introductory post is followed by a series that will examine accessibility, by travel time, to playgrounds in Wellington. Playgrounds are an important recreation resource and are typically accessed on foot. The posts will consider just how reasonable their pedestrian accessibility actually is.

The topic is narrow, but intended for covering concepts in depth. The following posts will cover:
- The impact of topology (hills) on travel times
- Modelling and visualising differences between suburbs
- The impact of adding new playgrounds on travel times. Do they make a difference to the suburban average?

The final post will close the series by examining how accessibility analyses can inform about the "liveability" of an urban environment. 
