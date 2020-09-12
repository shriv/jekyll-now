---
layout: archive 
permalink: /mini-investigations/
title: Mini Investigations
author_profile: false

description: "This little corner is a structured but miscellaneous collection of investigations. The aim is to make them quantitative, bite-sized and loosely connected."
toc: true
---
A little bit more of a description here. 

## Latest stories

<div class="grid__wrapper">
  {% assign collection = 'mini-investigations' %}
  {% assign posts = site[collection] | reverse %}
  {% for post in posts %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>
